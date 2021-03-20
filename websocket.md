### websocket

#### 属性

* readyState 当前连接状态
  * 0 connecting
  * 1 open
  * 2 closing
  * 3 closed

````js
let socket
// 心跳检测
let heartCheck = {
    timeout: 3000,
    serverTimeout: 4000,
    timeoutObj: null,
    serverTimeoutObj: null,
 	reset: function(){
        clearTimeout(this.timeoutObj)
    	clearTimeout(this.serverTimeoutObj)
    	this.start()
    },
    start: function(){
        let self = this;
        this.timeoutObj = setTimeout(funciton(){
        	socket.send("ws_heartbeat")
        	self.serverTimeoutObj = setTimeout(function(){
                socket.close()
            },self.serverTimeout)
        },this.timeout)
    }
}
export function wsconnect(){
    socket = new Websocket('ip');
    // 建立连接时 readyState=1
    socket.onopen= function(){
        if(socket.readyState===1){
            // 向服务器发消息
            socket.send('ws_connect')
            heartCheck.start()
        }
    }
    // 收到消息时
    socket.onmessage = function(messageEvent){
        let origindata = messageEvent?.data
        if(origindata){
            try{
                // 心跳重置，只要服务端有发消息就默认连接OK
                // if(data && data.data.msg === 'ws_heartbeat'){
                	heartCheck.reset()
               	 	//   return
                // }
            }catch{}
        }
    }
    // 发生错误时
    socket.onerror = function(){
        reconnect()
    }
    // 连接关闭时 readyState=3
    socket.onclose = function(){
        // 需保持一直连接
        wsconnect()
    }
}
// 断开重连
let lockReconnect = false
function reconnect(){
    if(lockReconnect) return
    lockReconnect = true
    // 没连上会一直重连，设置延迟避免请求过多
    setTimeout(()=>{
        wsconnect()
        lockReconnect = false
    },2000)
}
````

