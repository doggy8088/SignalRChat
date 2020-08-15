# SignalRChat

## 執行方式

1. 啟動網站

    ```sh
    dotnet run
    ```

2. 開啟瀏覽器到以下網址

    <https://localhost:5001/>

3. 開啟 F12 開發者工具並切換到 Console 頁籤

    執行以下命令，看 SignalR Hub 是否有傳回資料並呼叫 `ReceiveMessage` 事件的 Callback

    ```js
    connection.invoke("SendMessage", 'Tom', 'Hey, man!')
    ```

    ![image](https://user-images.githubusercontent.com/88981/90314956-1fa98a00-df4a-11ea-8119-21f616ff0ad7.png)

## 重點檔案

- `Hubs\ChatHub.cs`

    加入 `ChatHub` SignalR 服務

    ```cs
    using Microsoft.AspNetCore.SignalR;
    using System.Threading.Tasks;

    namespace SignalRChat.Hubs
    {
        public class ChatHub : Hub
        {
            public async Task SendMessage(string user, string message)
            {
                await Clients.All.SendAsync("ReceiveMessage", user, message);
            }
        }
    }
    ```

- `Startup.cs`

    將 SignalR 加入 ServiceCollection (DI 容器)

    ```cs
    services.AddSignalR();
    ```

    設定 `ChatHub` 的 Endpoint

    ```cs
    endpoints.MapHub<ChatHub>("/chatHub");
    ```

- `Pages\Shared\_Layout.cshtml`

    將 `@microsoft/signalr` 用戶端程式庫加入到網頁中，且必須出現在 `~/js/site.js` 之前。

    ```html
    <script src="https://cdnjs.cloudflare.com/ajax/libs/microsoft-signalr/3.1.7/signalr.js"></script>
    <script src="~/js/site.js" asp-append-version="true"></script>
    ```

- `wwwroot\js\site.js`

    將 SignalR 用戶端程式碼加入到這個檔案中，註冊 SignalR 事件、啟動 SignalR 連線、對伺服器發出要求。

    ```js
    var connection = new signalR.HubConnectionBuilder().withUrl("/chatHub").build();

    connection.on("ReceiveMessage", function (user, message) {
        alert(`Hi ${user}, you said: ${message}`)
    });

    connection.start().then(function () {
        connection.invoke("SendMessage", 'Will', 'Hello World').catch(function (err) {
            return console.error(err.toString());
        });
    }).catch(function (err) {
        return console.error(err.toString());
    });
    ```