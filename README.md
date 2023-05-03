Download Link: https://assignmentchef.com/product/solved-cs-522-assignment-2-chat-app-ui
<br>
Develop an app that is backward compatible with the Q (Android 10.0, API 29) version of the Android platform for your submission for this assignment (also set minSdkVersion to 22).  Set the targetSdkVersion and the compileSdkVersion to API 29.  Test your assignment on emulated devices of your choice.

You will complete two apps so that they will speak to each other peer-to-peer using UDP sockets.  These apps are very simple and violate some of the design guidelines for Android apps, such as not performing network communication on the main thread.  We will see later how to fix this.

You are provided with two apps.  ChatServer has a single button, Next.  When you press it, it waits to receive a UDP datagram packet from the network on a designated port, and appends the message in the packet to a list of messages that are displayed on the screen.  ChatClient has a message editor window and a button, Send.  When you press SEND, the contents of the message edit window are sent to the server.

On real devices, the client and server apps are intended to communicate by broadcasting over a Bluetooth or WIFI network that they are both connected to.  Since the emulator does not implement the WIFI<a href="#_ftn1" name="_ftnref1"><sup>[1]</sup></a> or Bluetooth stacks, for the sake of this assignment we will just have the server bind to a server port, and the client then sends messages to that port.  For machine addressing, the client will send to the loopback interface for the host machine on which both the client and server AVDs execute, and redirect messages to the server UDP port on the host machine to the corresponding port on the server AVD.

Every message that the client sends should have a name prepended to it (at the front of the message).  On the server, the name is separated from each incoming requests. Currently the name is specified in the client UI.  Eventually, in a future assignment, you will add an activity that allows this name to be defined as an app preference.

In addition, provide another UI (accessible using the options menu from the action bar) for displaying a list of the peers.  This UI just lists peers by name in a list view.  The list of peers is maintained in the main chat server activity, and passed to the subactivity for displaying the list of peers as a parcelable array list extra.

If the user selects one of these peers, then provide in a third UI information about that peer (their currently known IP address, and the last time that a message was received from that user).  The peer information is passed as a parcelable extra to this list.

To prepare for future assignments, and get some practice with parcels and the Parcelable interface, define entity classes that contain the interesting state about messages and chat peers. Define these in an entities subpackage respectively of your app.  You have two kinds of entities: a message and a peer (message sender):

public class Message implements Parcelable {    public long id;    public String messageText;    public Date timestamp;    public String sender;    public long senderId;

}

public class Peer implements Parcelable {    public long id;    public String name;    public Date timestamp;    public InetAddress address;      public int port;

}




All of these entity objects should implement the Parcelable interface.  This involves writing a method that writes the state of the entity object to an output parcel, and a constructor that reconstitutes the state of an entity from an input parcel.  Also complete the implementation of the CREATOR static field for the entity class, the implements the

Parcel.Creator&lt;Peer&gt; interface.  Although not necessary for this assignment, you should also complete the implementation of the Message entity class, that also implements the Parcelable interface.  The library you are provided with includes utilities that you will find helpful: DateUtils.readDate and DateUtils.writeDate for dates, and InetAddressUtils.readAddress and InetAddressUtils.writeAddress for IP addresses. The latter uses the Guava library to avoid DNS lookup in the standard InetAddress constructors!

You should follow this strategy to get the client and server to talk to each other:

<ol>

 <li>Create separate virtual devices (AVDs) for the client and server. Let’s say you call these ChatClient and ChatServer, respectively.</li>

 <li>In the view for the ChatClient project, lick on the “app” drop-down menu and pick “Edit Configurations”:</li>

</ol>

For the Device Target Option, choose Emulator and then pick ChatClient as the device to run the application on:

This is to make sure that the client app runs on the ChatClient device.

<ol start="3">

 <li>Similarly, make sure that the server app runs on the ChatServer device.</li>

 <li>Run the server and client chat apps. Assuming that you ran the server first, followed by the client, the corresponding AVDs have administrative port numbers 5554 and 5556, respectively.</li>

 <li>The apps are not yet communicating, because they are running on their own network stacks. In particular, the server has bound to a UDP port on the server guest machine.  You now need to bind that to a UDP port on the host machine upon which you’re running these two AVDs.  This is easy to do.  The server binds to UDP port 6666 on its guest Ethernet interface (10.0.2.15), and the client will try to send messages to port 6666 on the host loopback interface (10.0.2.2 on the AVD, 127.0.0.1 on the host machine).  Use the adb command in the platformtools subdirectory of the Android installation to perform this redirection<a href="#_ftn2" name="_ftnref2"><sup>[2]</sup></a>, by typing this line in the OS (bash or Windows) shell:</li>

</ol>

adb –s emulator-5554 forward tcp:6666 tcp:6666

<ol start="6">

 <li>You may want to change the first port (the first 6666, the chat server port on the host machine) if you are running on a machine where another client has bound to that address. You don’t need to change the second 6666 since that is the server port on the guest device.</li>

</ol>




You can find out more about network redirection here:  <u>https://developer.android.com/studio/run/emulator-networking.html#connecting</u>







The client and server apps include some functionality in an external module, imported as an AAR file<a href="#_ftn3" name="_ftnref3"><sup>[3]</sup></a>.  The settings.gradle file should include this line:




include ‘:app’, ‘:cs522-library’




The build.gradle file for the app should include these dependencies:




implementation project(“:cs522-library”)

implementation ‘com.google.guava:guava:28.1-android’

<a href="#_ftnref1" name="_ftn1">[1]</a> The emulator with a system software stack for version 25 or greater does support WIFI, but we will work with point-to-point communication rather than debugging WIFI broadcast.

<a href="#_ftnref2" name="_ftn2">[2]</a> Connecting to the AVD via telnet and using the redir command is not always reliable.  Also, we are forwarding TCP rather than UDP, and using a class DatagramSendReceive that provides a datagram socket API wrapped around TCP, to work around an apparent bug in the emulator.

<a href="#_ftnref3" name="_ftn3">[3]</a> To add this module into an existing project, click File | New | New Module and select Import .JAR/.AAR Package.