<script setup>
import { ref } from 'vue'

defineProps({
  msg: String,
})

const count = ref(0)
</script>

<template>
  <h1>{{ msg }}</h1>
  <h2>{{timeout}}</h2>
  <h2>{{hello}} </h2>

  <div class="card">
    <button type="button" @click="count++">count is {{ count }}</button>
    <p>
      Edit
      <code>components/HelloWorld.vue</code> to test HMR
    </p>
  </div>

  <form>


  <p>
    <label for="name">Name</label>
    <input
      id="name"
      v-model="name"
      type="text"
      name="name"
    >
  </p>

  <button type="button" @click="submitForm">提交</button>

</form>

  <p>
    Check out
    <a href="https://vuejs.org/guide/quick-start.html#local" target="_blank"
      >create-vue</a
    >, the official Vue + Vite starter
  </p>
  <p>
    Install
    <a href="https://github.com/vuejs/language-tools" target="_blank">Volar</a>
    in your IDE for a better DX
  </p>
  <p class="read-the-docs">Click on the Vite and Vue logos to learn more</p>
</template>

<style scoped>
.read-the-docs {
  color: #888;
}
</style>

<script>
	export default {
		// components: { VueQr },
		data() {
			return {
				lockReconnect: false, //是否真正建立连接
				timeout: 58 * 1000, //58秒一次心跳
				timeoutObj: null, //心跳心跳倒计时
				serverTimeoutObj: null, //心跳倒计时
				timeoutnum: null, //断开 重连倒计时
        hello:"123",
        name:"123",
			};
		},
		created() {
			this.initWebSocket();
		},
		destroyed() {
			this.websock.close(); //离开路由之后断开websocket连接
		},
		methods: {
      submitForm (e) {
       this.websock.send(this.name);
      },
			currentTime() {
				setInterval(this.formatDate, 500);
			},
			initWebSocket() {
				//初始化weosocket
				// const wsuri = "wss://xxxxxxx/xx/xx/" + this.roomId;
        const wsuri ="ws://127.0.0.1:8080"
				this.websock = new WebSocket(wsuri);
				// 客户端接收服务端数据时触发
				this.websock.onmessage = this.websocketonmessage;
				// 连接建立时触发
				this.websock.onopen = this.websocketonopen;
				// 通信发生错误时触发
				this.websock.onerror = this.websocketonerror;
				// 连接关闭时触发
				this.websock.onclose = this.websocketclose;
			},
			// 连接建立时触发
			websocketonopen() {
				//开启心跳
				this.start();
				//连接建立之后执行send方法发送数据
				// let actions = {"room":"007854ce7b93476487c7ca8826d17eba","info":"1121212"};
				// this.websocketsend(JSON.stringify(actions));
			},
			// 通信发生错误时触发
			websocketonerror() {
				console.log("出现错误");
				this.reconnect();
			},
			// 客户端接收服务端数据时触发
			websocketonmessage(e) {
				console.log(e.data);
        this.hello = e.data
				//收到服务器信息，心跳重置
				this.reset();
			},
			websocketsend(Data) {
				//数据发送
				this.websock.send(Data);
			},
			// 连接关闭时触发
			websocketclose(e) {
				//关闭
				console.log("断开连接", e);
				//重连
				this.reconnect();
			},
			reconnect() {
				//重新连接
				var that = this;
				if (that.lockReconnect) {
					return;
				}
				that.lockReconnect = true;
				//没连接上会一直重连，设置延迟避免请求过多
				that.timeoutnum && clearTimeout(that.timeoutnum);
				that.timeoutnum = setTimeout(function () {
					//新连接
					that.initWebSocket();
					that.lockReconnect = false;
				}, 5000);
			},
			reset() {
				//重置心跳
				var that = this;
				//清除时间
				clearTimeout(that.timeoutObj);
				clearTimeout(that.serverTimeoutObj);
				//重启心跳
				that.start();
			},
			start() {
				//开启心跳
				console.log("开启心跳");
				var self = this;
				self.timeoutObj && clearTimeout(self.timeoutObj);
				self.serverTimeoutObj && clearTimeout(self.serverTimeoutObj);
				self.timeoutObj = setTimeout(function () {
					//这里发送一个心跳，后端收到后，返回一个心跳消息，
					if (self.ws.readyState == 1) {
						//如果连接正常
						// self.ws.send("heartCheck"); //这里可以自己跟后端约定
					} else {
						//否则重连
						self.reconnect();
					}
					self.serverTimeoutObj = setTimeout(function () {
						//超时关闭
						self.ws.close();
					}, self.timeout);
				}, self.timeout);
			},
		},
		mounted() {
			this.currentTime();
		},
		// 销毁定时器
		beforeDestroy() {
			if (this.formatDate) {
				clearInterval(this.formatDate); // 在Vue实例销毁前，清除时间定时器
			}
		},
	};
</script>
