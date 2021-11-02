# SimpleChatAppWithSignalR

This tutorial teaches the basics of building a real-time app using SignalR. 

1- Create a new ASP.NET Core Web Application project.

2- Then add the SignalR client library:
- In Solution Explorer, right-click the project, and select Add > Client-Side Library
- In the Add Client-Side Library dialog, for Provider select unpkg
- For Library, enter @microsoft/signalr@latest
- Select Choose specific files, expand the dist/browser folder, and select signalr.js and signalr.min.js
- Set Target Location to wwwroot/js/signalr/, and select Install

3- Create a SignalR hub
- In the SignalRChat project folder, create a Hubs folder.
- In the Hubs folder, create a ChatHub.cs file with the following code:
```
    public class ChatHub : Hub
    {
        public async Task SendMessage(string user, string message)</div>
        {
            await Clients.All.SendAsync("ReceiveMessage", user, message);
        }
    }
```

4- Configure SignalR server  in Startup.cs class.
```
public void ConfigureServices(IServiceCollection services)
        {
            services.AddRazorPages();
            services.AddSignalR();
        }
 ```
 And
 ```
 public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
        {
            ...
            app.UseEndpoints(endpoints =>
            {
                endpoints.MapRazorPages();
                endpoints.MapHub<ChatHub>("/chatHub");
            });
        }
 ```
 
 5- Replace the content in Pages\Index.cshtml with the following code:
 ```
 @page

<input type="text" id="userInput" placeholder="User Name"/>
<input type="text" id="messageInput" placeholder="User Message" />
<input type="button" id="sendButton" value="Send Message" />

<ul id="messagesList"></ul>

<script src="~/js/signalr/dist/browser/signalr.js"></script>
<script src="~/js/chat.js"></script>
 ```
 
 6- Create a chat.js file in the wwwroot/js folder with the following code:
 ```
 "use strict";

var connection = new signalR.HubConnectionBuilder().withUrl("/chatHub").build();

//Disable send button until connection is established
document.getElementById("sendButton").disabled = true;

connection.on("ReceiveMessage", function (user, message) {
    var li = document.createElement("li");
    document.getElementById("messagesList").appendChild(li);
    // We can assign user-supplied strings to an element's textContent because it
    // is not interpreted as markup. If you're assigning in any other way, you 
    // should be aware of possible script injection concerns.
    li.textContent = `${user} says ${message}`;
});

connection.start().then(function () {
    document.getElementById("sendButton").disabled = false;
}).catch(function (err) {
    return console.error(err.toString());
});

document.getElementById("sendButton").addEventListener("click", function (event) {
    var user = document.getElementById("userInput").value;
    var message = document.getElementById("messageInput").value;
    connection.invoke("SendMessage", user, message).catch(function (err) {
        return console.error(err.toString());
    });
    event.preventDefault();
});
 ```

7- Finally run the project and copy the URL from the address bar, open another browser instance or tab, and paste the URL in the address bar and start chat.
