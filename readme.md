### Sample UDP Broadcast Server
This simulates the echo webserver broadcast

Run with Node.js
```
node DesktopUDPBroadcaster.js
```
Or with Coffeescript
```
coffee DesktopUDPBroadcaster.coffee
```
### IOS Sample Project
Simply open the XCWorkspace in XCode 7.2.1+ and compile onto an iPhone.
*Note: if you try to run the IOS Sample Project in the IOS simulator on the same computer that is running the Broadcast server, you will get a "UDP Address in use error".  Just run the project or server on another computer on the same network*
```
open T100Scanner.xcworkspace
```
## To build your own Swift project from scratch

### Install Cocoapod
Add CocoaAsyncSocket to your Podfile
```
use_frameworks!
pod 'CocoaAsyncSocket'
```
Then run
```
pod install
```
### Import CocoaAsyncSocket
Add this to the beginning of your Swift View Controller
```
import CocoaAsyncSocket
```
### Add **GCDAsyncUdpSocketDelegate** to your view controller
```
// EXAMPLE
class ViewController: UIViewController, GCDAsyncUdpSocketDelegate, UITableViewDataSource, UITableViewDelegate {
```
### Set global variable
This is the Diciontary that stores all of the active echo servers on your subnet.
```
var servers = Dictionary<String, String>()
```
To clear the dictionary, simply remove all items
```
self.servers.removeAll()
```
### Simple scanning function
To start scanning for echo servers simply create a GCDAsyncUdpSocket, bind it to a port, and enable the broadcast
```
func startScanning() {
    let ssdpAddres          = "224.0.0.1"
    let ssdpPort:UInt16     = 1612
    let ssdpSocket = GCDAsyncUdpSocket(delegate: self, delegateQueue: dispatch_get_main_queue())
    try! ssdpSocket.bindToPort(ssdpPort)
    try! ssdpSocket.enableBroadcast(true)
    try! ssdpSocket.beginReceiving()
    try! ssdpSocket.joinMulticastGroup(ssdpAddres)
}
```

### udpSocket Delegate Method
This parses the JSON received over UDP and inserts the server ip addresses into the servers global variable
```
func udpSocket(sock: GCDAsyncUdpSocket!, didReceiveData data: NSData!, fromAddress address: NSData!, withFilterContext filterContext: AnyObject!) {
    let json = try! NSJSONSerialization.JSONObjectWithData(data, options: NSJSONReadingOptions.MutableContainers)
    if let result = json as? Dictionary<String, String> {
        if let server:String = result["server"], status:String = result["status"] {
            if status == "true" {
              self.servers[server] = "Recording"
            }
            if status == "false" {
              self.servers[server] = "Not Recording"
            }
        }
        //self.tableView.reloadData()
    }
}
```
