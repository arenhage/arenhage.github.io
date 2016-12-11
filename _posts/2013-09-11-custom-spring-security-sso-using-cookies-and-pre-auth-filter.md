---
layout: post
title: Custom Spring Security SSO using cookies and pre-auth filter
date: 2013-09-11 11:55
author: arenhage
comments: true
categories: [Authentication, cookiePreAuthFilter, grails, Grails/Groovy, Home, Single sign-on, Spring Security]
---

Versions:
<ul>
	<li><strong>Grails v2.1.1</strong></li>
	<li><strong>MongoDB v.2.2.5</strong></li>
	<li><strong>Spring Security Core plugin 1.2.7.3</strong></li>
	<li><strong>Cookie Plugin v.0.4</strong></li>
</ul>

When using the Spring-security core plugin, we have the possibility of utilizing SSO for our spring security applications by extension plugins (<a href="http://grails.org/plugin/spring-security-core">http://grails.org/plugin/spring-security-core</a>) . However, by using these extension plugins we need to introduce third party logic for our application in order to authenticate our users.

<!--more-->

In my scenario, i have a couple of applications that i want to deploy to the same servlet container (e.g Tomcat) and provide a higher level of SSO to all these applications by using the spring security plugin in the grails framework. I wanted to come up with some solution where i could offer SSO to all my applications considered virtually to be in the same login session, without the interference of any third party entities.

Spring security offers so called remember-me cookies (<a href="http://grails-plugins.github.io/grails-spring-security-core/docs/manual/guide/9%20Authentication.html#9.3%20Remember-Me%20Cookie">http://grails-plugins.github.io/grails-spring-security-core/docs/manual/guide/9%20Authentication.html#9.3%20Remember-Me%20Cookie</a>), basically offering the possibility for users not be required to provide its credentials for each session. My first attempt was to, for each application provide identical properties such as <em>rememberMe.key</em> and <em>rememberMe. tokenValiditySeconds </em>in order for  the applications to pick up the same remember-me cookie as the first application had created. <a href="http://static.springsource.org/spring-security/site/docs/3.0.x/reference/remember-me.html">http://static.springsource.org/spring-security/site/docs/3.0.x/reference/remember-me.html</a>. This did not work since the cookie that was written to the browser had its path specified to what application it was produced by, i tried to override this by in my tomcat define <em>emptySessionPath</em> my context.xml ( <a href="http://tomcat.apache.org/migration-7.html#Session_cookie_configuration">http://tomcat.apache.org/migration-7.html#Session_cookie_configuration</a> ) to write all cookies to "/", but this did not have any affect on whether the spring-security plugin would pick up the cookie or not. Maybe i have missed something, please let me know if i did!

The solution i came up with was to create my own cookies and in the filter chain add a custom <em>cookiePreAuthFilter</em> providing information to an underlying preAuthProvider. This worked nicely in my setup for two distinct reasons:
<ul>
	<li>I did not need to change any of my prior implemented auth-providers since the cookie authentication was simply just part of the preAuthProvider added in the beginning of the provider chain</li>
	<li>Any application that would need to utilize this SSO implementation simply needed to know the name of the cookie and security key used in the cookie <em>one way hash.</em></li>
</ul>
<span style="text-decoration:underline;">The implementation</span>

For future reference, how we have defined the cookie content will answer some questions on why we do certain stuff.

When we have a successful login , we write a cookie to the browser with some content. The cookie will be identifiable by a constant name commonly known for any application wishing to implement the SSO. The content of the cookie will be defined as follows:

<blockquote>base64(username + ":" + expirationTime + ":"  + sha256(username + ":" + expirationTime + ":" encoded(password) + ":" + key))</blockquote>

(Basically the same as for the spring-security remember cookies ( <a href="http://docs.spring.io/spring-security/site/docs/3.0.x/reference/remember-me.html">http://docs.spring.io/spring-security/site/docs/3.0.x/reference/remember-me.html</a> ))

In my <strong>config.groovy</strong> i have defined my list of authProviders

```java
grails.plugins.springsecurity.providerNames = [
  'preAuthProvider',
  'rememberMeAuthenticationProvider',
  'ldapAuthProvider',
  'daoAuthenticationProvider',
  'anonymousAuthenticationProvider']
```

the providers beans are defined in our <strong>resources.groovy </strong>(i'm only describing the cookie providers here so i wont include everything)

```java
userDetailsServiceWrapper(UserDetailsByNameServiceWrapper) {
  userDetailsService = ref('userDetailsService')
}
cookiePreAuthFilter(CookiePreAuthFilter) {
  authenticationManager = ref("authenticationManager")
  springSecurityService = ref("springSecurityService")
  cookieService = ref("cookieService")
  grailsApplication = ref("grailsApplication")
}
preAuthProvider(org.springframework.security.web.authentication.preauth.PreAuthenticatedAuthenticationProvider) {
  preAuthenticatedUserDetailsService = ref("userDetailsServiceWrapper")
}
```
<ul>
	<li>The <strong>userDetailsServiceWrapper </strong>is used to allow authentication by only username, this will be explained below why we do this.</li>
	<li>The <strong>cookiePreAuthFilter </strong>is jacked in the filter chain to pickup the cookie from the request and to provide the cookie content to the pre-auth provider.</li>
	<li>The <strong>preAuthProvider </strong>will if successful grant access to the application and create the security token.</li>
</ul>
In our <strong>Bootstrap.groovy  </strong>we want to add cookiePreAuthFilter the filter to the filterchain. We add it to the filter position for PRE_AUTH since we want to jack it in in the beginning :

```java
SpringSecurityUtils.clientRegisterFilter('cookiePreAuthFilter', SecurityFilterPosition.PRE_AUTH_FILTER)
```

So we have now jacked in our filter, now we just have to define what it should do:

```java
class CookiePreAuthFilter extends AbstractPreAuthenticatedProcessingFilter {
  def springSecurityService
  def grailsApplication
  def cookieService

  protected Object getPreAuthenticatedCredentials(HttpServletRequest arg0) {
    return ""
  }

  protected Object getPreAuthenticatedPrincipal(HttpServletRequest arg0) {
    //in what ever manner you feel suitable, get a hold of the cookieKey, name and timeExpire
    def cookieKey = grailsApplication.config.basic.cookieKey
    def cookieName = grailsApplication.config.basic.cookieName
    def cookieTimeExpire = grailsApplication.config.basic.cookieTimeExpire
    def splitContent = new String(cookieService.get(cookeName)?.decodeBase64()).split(":")   //we get a hold of the username we are authenticating
    if(splitContent && splitContent.size() > 0) {
      //In our database, we get the user with the username from the cookie content.
      def userInstance = User.findByUsername(splitContent[0])
      //we generate a hash based on the same algorithm as described before with the user information from the user in the database
      def hashFromContent = GenerateHashHelper.generateHash(userInstance.username, userInstance.password, cookieTimeExpire, cookieKey)
      //check if they are the same, in that case we return the username..
      if(hashFromContent == splitContent[1]) {
        return userInstance.username
      }
    }
    return "";
  }
}
```

When <strong>getPreAuthenticatedPrincipal </strong>returns, it will return a value to the <strong>preAuthProvider</strong> in the authprovider chain. Since we have defined our <strong>preAuthProvider</strong> in resources.groovy as:

<strong>preAuthenticatedUserDetailsService = ref("userDetailsServiceWrapper")</strong>

Using the <em><strong>UserDetailsByNameServiceWrapper </strong></em>defined for the <strong>userDetailsServiceWrapper</strong><em> </em>we are able to grant access based on username returned by the preAuthFilter only when we were able to validate the <em>content hash.</em>

<span style="text-decoration:underline;">To sum up</span>
<ul>
	<li>Obviously we are vulnerable to cookie theft, this is of course a factor.</li>
	<li>If we have an external authentication provider such as an LDAP provider, this implementation will still work if we persist the username with some complex "random" password in our local database.</li>
	<li>It is easy to add new applications to the SSO session by just knowing the commonly known elements (<em>cookieKey</em>, <em>cookieName</em>) for the key hash.</li>
	<li>Each application wanting to be a part of the SSO need to have access to the database containing the user/credentials, or have a copy of the information in its own database or persistence object of some sort.</li>
	<li>We do not need include any ticket authority or third party software maintaining our SSO users.</li>
</ul>
