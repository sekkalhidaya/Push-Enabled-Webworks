Push-Enabled-Webworks
=====================

How the Push Enabled (or the BlackBerry device) receive pushes from the push Initiator, Webworks HTML5.

What do you need as authentification data to built the Push Enabled is: the Initiator PPG url, password, username and PPG url for the client side (for push enabled), provided by Rim after your subscription for the Push Service.

  1. First you need to build a channel to communicate between the device (Push Enabled) and Server (Push Initiator),
     Create a channel to the PPG (Push Proxy Getway) that include the create channel Callback, this one will return a token that refer to a device and the push enabled application. 

        sample.pushcapture.pushService.createChannel(sample.pushcapture.createChannelCallback);

        sample.pushcapture.constructor.prototype.createChannelCallback = function(result, token) {
          if (result == blackberry.push.PushService.SUCCESS) {
              if (sample.pushcapture.usesdkaspi) {
                  // The Push Service SDK is being used, so attempt to subscribe to the Push Initiator
                     sample.pushcapture.subscribeToPushInitiator(token);
              } else {
                  // The Push Service SDK is not being used, jump to displaying a success
                     sample.pushcapture.successfulRegistration();
              }
           }
        }
  
  
  The token that we already created (that refers to the device and the Push Enabled application) will be send to the Push Initiator so that the Push Initiator can use the token to send pushes to the device that will be recieved buy the push enabled application. 

      sample.pushcapture.constructor.prototype.subscribeToPushInitiator = function(token) {
        document.getElementById("progressinfo").innerHTML = "Subscribing to Push Initiator...";
        var type;
        if (sample.pushcapture.usingpublicppg) {
            type = "public";
        } else {
            type = "bds";
        }
        var username = document.getElementById("reguserid").value.trim();
        var password = document.getElementById("regpwd").value.trim();
        var address = token;
        var osversion = blackberry.system.softwareVersion;
        var model = blackberry.system.hardwareId;
     
        var params = "appid=" + encodeURIComponent(sample.pushcapture.appid)
                      + "&";
        params += "address=" + address + "&";
        params += "osversion=" + osversion + "&";
        params += "model=" + model + "&";
        params += "username=" + encodeURIComponent(username) + "&";
        params += "password=" + encodeURIComponent(password) + "&";
        params += "type=" + type;
     
        var subscribeUrl = sample.pushcapture.piurl + "/subscribe?" + params;
        var xmlHttp = new XMLHttpRequest();
     
        xmlHttp.open("GET", subscribeUrl);
     
        xmlHttp.onreadystatechange = function() {
            if (xmlHttp.readyState == 4) {
                var status = xmlHttp.status;
                var returnCode = xmlHttp.responseText;
     
                sample.pushcapture.pushInitiatorSubscribeHandler(status,
                       returnCode);
            }
        };
     
         xmlHttp.send();
      };
  
  2. Create a push service using the PPG urls and the authentification dada provided by Rim, 

        PushCapture.prototype.createPushService = function() {
            var ops;
            if (sample.pushcapture.usingpublicppg) {
                // Consumer application using public push
                ops = {
                    invokeTargetId : sample.pushcapture.invokeTargetIdPush,
                    appId : sample.pushcapture.appid,
                    ppgUrl : sample.pushcapture.ppgurl
                };
            } else {
                // Enterprise application using enterprise push
                if (sample.pushcapture.usesdkaspi) {
                    // If we're using the Push Service SDK for our Push Initiator
                    // implementation, we will have specified our own application ID to use
                    ops = {
                        invokeTargetId : sample.pushcapture.invokeTargetIdPush,
                        appId : sample.pushcapture.appid
                    };
                } else {
                    ops = {
                        invokeTargetId : sample.pushcapture.invokeTargetIdPush
                    };
                }
            }
    
            blackberry.push.PushService
                    .create(ops, sample.pushcapture.successCreatePushService, sample.pushcapture.failCreatePushService,
                            sample.pushcapture.onSimChange, sample.pushcapture.onPushTransportReady);
       };



  3. Call the Invocation methods and API, to invoke the BlackBerry device: 

          /**
          * Invoke target ID for receiving new push notifications.
          */
          this.invokeTargetIdPush = "sample.pushcapture.invoke.push";
          
          /**
          * Invoke target ID when clicking on a notification in the BlackBerry Hub opens the app.
          */
          this.invokeTargetIdOpen = "sample.pushcapture.invoke.open";


  4. Build your xml file (config.xml):
         Add a reference to the blackberry.push feature.
         Add a feature tag for blackberry.invoked. This feature tag relates to the handling of invoke events. A push message arrives at your application as an invoke event.
         If you are using the Push Service with the BlackBerry Internet Service, add a permission statement for the Push Service so that your application can work with push messages.
         Add an entry for the invoke events that your application receives for push messages.

          <feature id=”blackberry.push” />
          <feature id=“blackberry.invoked” />
          <rim:permissions>
               <rim:permit system=”true”>_sys_use_consumer_push</rim:permit>
          </rim:permissions>
          <!-- config.xml -->
          <!-- Need to put an invoke entry here for push. -->    
          <!-- The id here must match the invokeTargetId passed in -->
          <!-- one of the ops to blackberry.push.PushService.create. -->    
           <rim:invoke-target id="sample.pushreceiver.invoke.push">      
             <type>APPLICATION</type>      
             <filter>        
               <action>bb.action.PUSH</action>
               <mime-type>application/vnd.push</mime-type>      
             </filter>    
           </rim:invoke-target>
 
  
  5. The push enabled is a client side applicatoin that you need to load it in your device to be able to test it and make it work.  
     After that your Push Enabled will be ready to receive pushes from the Push Initiator and Invoke your device (you will do that by going to you Server side application and send pushes).   
     If you didn't built yet your server side application (Initiator Push) you can visit: https://github.com/sekkalhidaya/Push-Initiator-Webworks/edit/master/README.md 
    
 
For more clarification about the Push Enabled you can visit: https://github.com/blackberry/BB10-WebWorks-Samples/tree/master/pushCapture     
For more clarification about the invoke methode (invication) you can visit: http://developer.blackberry.com/html5/api/blackberry.invoke.html 
