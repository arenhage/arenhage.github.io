---
layout: post
title: Mule and FTP, downloading multiple files from within a Callable component
date: 2013-10-11 10:58
author: arenhage
comments: true
categories: [ESB, FTP, Home, Java, Mule, MuleESB]
---
Versions:
<ul>
	<li><strong>MuleESB v.3.2.1</strong></li>
</ul>
This is just a solution i wrote in order to dynamically download multiple files from an FTP server programmatically from within a Mule Callable Component.

This is most likely not the most kosher way of getting multiple files from an FTP, however it did give me the satisfactory results since i was able to selectively choose which files to download and one by one send out in the flow construct.

<!--more-->

The tricky part was that i wanted to get files with some specific prefix.. e.g  we are only interested in the files starting with the letter 'A'. So for a file list with 3 files  A-X.xml, B-Y.json, A-XX.xml, we only want to download file A-X and A-XX.

The first part is implementing the Callable interface which provides us with the onCall(MuleEventContext eventContext) method. In this example we are providing the prefix in the payload fetching it from the muleMessage in the eventContext. We then get a hold of the filelist on the ftp by using a FTPClient lib and issuing <strong>listNames.</strong>

For any file matching our prefix pattern, we get the file with the <strong>getFile </strong>method and send it out on the flow construct to our destination flow, in this case to a vm endpoint name <strong>processing.</strong>

```java
public class GetFiles implements Callable {
  public Object onCall(MuleEventContext eventContext) throws Exception {
    // Get the filename
    Object filenameprefix = (Object) eventContext.transformMessage(Object.class);
    MuleEventContext eventCtx = RequestContext.getEventContext();

    MuleMessage msg = null;
    //iterate through the filelist on the ftp server.
    for(String filename : getFileList()) {
      if(filename.matches(filenameprefix+".*")) {
        //if we find a file with a filename matching our regex
        msg = getFile(eventCtx, filename);  //get the file from the ftp
        msg.setOutboundProperty(FileConnector.PROPERTY_ORIGINAL_FILENAME, filename);
        msg.setOutboundProperty(FileConnector.PROPERTY_FILENAME, filename);
        if(msg.getPayloadAsString() != null || msg.getPayloadAsString() != "") {
          eventCtx.sendEvent(msg, "vm://processing");  //send the message to the destination endpoint
        }
        else {
          throw new DefaultMuleException("Payload corrupt");
        }
      }
    }
    return msg;
  }

  public MuleMessage getFile(MuleEventContext eventCtx, String filename) throws MuleException {
    //build an ftp endpoint
    EndpointBuilder ftpEndpointBuilder = RequestContext.getEvent().getMuleContext().getRegistry()
      .lookupEndpointFactory().getEndpointBuilder("ftp://" + getFtp_user() + ":" + getFtp_password() + "@" + getFtp_location()
         + 	":" + getFtp_port() + "/" + getFtp_path());
      //get a specified file based on the filename. Add a message processor to the ftpEndpointBuilder with a FileNameWildcardFilter.
      ftpEndpointBuilder.addMessageProcessor(new org.mule.routing.MessageFilter(
        new org.mule.transport.file.filters.FilenameWildcardFilter((filename))));
      MuleMessage msg = eventCtx.requestEvent(ftpEndpointBuilder.buildInboundEndpoint(), timeout); //fetch the file
      //if we could not find the file, we throw back an error
      if (msg == null) {
        throw new DefaultMuleException("File -- " + filename + " -- not found");
      }
      return msg;
  }

  /*
  * Get a list of all files available at the specified path
  */
  public String[] getFileList() {
    FTPClient client = new FTPClient();
    String[] names = null;
    try {
      client.connect(getFtp_location(), Integer.parseInt(getFtp_port()));
      client.login(getFtp_user(), getFtp_password());
      names = client.listNames(getFtp_path()); //list the names of all files on the ftp.
      client.logout();
    } catch (IOException e) {
      e.printStackTrace();
    } finally {
      try {
        client.disconnect();
      } catch (IOException e) {
        e.printStackTrace();
      }
    }
    return names; //return the list of names
  }
}
```
