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
    
    
    
