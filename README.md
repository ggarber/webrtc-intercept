# WebRTC Interceptor

When using some third-party WebRTC platforms or libraries sometimes we would like to be able to customize the way WebRTC is used by those libraries and it is not allowed with the official API.

In those cases it is easy to override the RTCPeerConnection constructor to be able to create the PeerConnection instances ourselves and to intercept any interesting method in those instances.

Contributions are more than welcomed.  Write your ideas as github issues or make a Pull Request to add new examples.

## Basic overriding

This is the basic code you  need to copy and paste to intercept the PeerConnections created inside your library/framework/platform.


```
(function() {
  var origPeerConnection = window.RTCPeerConnection || window.webkitRTCPeerConnection || window.mozRTCPeerConnection;
  if (origPeerConnection) {
    var newPeerConnection = function(config, constraints) {
        console.log('PeerConnection created with config', config);

        // Add here the specific logic you need
        // You can see some examples for specific use cases in the next sections of this document 

        return new origPeerConnection(config, constraints);
    }

    if (origPeerConnection === window.RTCPeerConnection) {
      window.RTCPeerConnection = peerconnection;
      window.RTCPeerConnection.prototype = origPeerConnection.prototype;
    } else if (origPeerConnection === window.webkitRTCPeerConnection) {
      window.webkitRTCPeerConnection = peerconnection;
      window.webkitRTCPeerConnection.prototype = origPeerConnection.prototype;
    } else if (origPeerConnection === window.mozRTCPeerConnection) {
      window.mozRTCPeerConnection = peerconnection;
      window.mozRTCPeerConnection.prototype = origPeerConnection.prototype;
    }
  }
})();
```

In most of the cases it is very important that you call this code before loading the third party library.   Otherwise it could be too late to intercept the PeerConnection constructor.

## Enable DSCP

One of the use cases I needed at some point is to enable DSCP in WebRTC (availabe only in Chrome with the proprietary constraint googDscp)

```
    var newPeerConnection = function(config, constraints) {
        constraints = constraints || {};
        constraints.optional = constraints.optional || [];
        constraints.optional.push({ googDscp: true });
        return new origPeerConnection(config, constraints);
    }
```

## Force relay transports

Other common use case is to force WebRTC to use relay candidates for privacy or route optimization reasons.

```
    var newPeerConnection = function(config, constraints) {
        config.iceTransportPolicy = 'relay';
        return new origPeerConnection(config, constraints);
    }
```

## Use your own ICE Servers

In some cases maybe you need to override the ICE servers used by default in the library/platform you use.   A specific example could be to try to use the TURN servers in your corporate network to traverse a very restrictive enterprise firewall.  You can easily do it this way:

```
    var newPeerConnection = function(config, constraints) {
        config.iceServers = [ {urls: 'turn:10.10.10.10'} ];
        return new origPeerConnection(config, constraints);
    }
```

## Grab extra stats

Sometimes you want to get your own statistics from PeerConnections for analytics or debugging purposes:

```
    var newPeerConnection = function(config, constraints) {
        var pc = new origPeerConnection(config, constraints);
        pc.getStats().then((stats) => {
        });
        return pc;
    }
```


## Send DTMF

This is a very strange use case but still a posibility.

```
    var newPeerConnection = function(config, constraints) {
        var pc = new origPeerConnection(config, constraints);
        // Store it in a global place
        // Be careful if you are using multiple peerconnections at the same time
        window.pc = pc;
        return pc;
    }

    function sendDtmf(tones) {
      window.pc.createDtmfSender().insertDtmf(tones);
    }
```
