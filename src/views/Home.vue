<template>
  <div class="container">
    <!-- 接收窗口 -->
    <div class="recv-area">
      <el-input
        type="textarea"
        v-model="recvOption.text"
        style="height: 100%;"
        readonly
        resize="none"
        id="recv-window"
      ></el-input>
    </div>
    <!-- 功能区 -->
    <div class="function-area">
      <el-button
        type="danger"
        size="mini"
        @click="onClickRecvClear"
      >清除接收窗口</el-button>
      <div>
        <el-checkbox v-model="recvOption.hex">HEX显示</el-checkbox>
        <el-checkbox v-model="recvOption.timestamp">加时间戳</el-checkbox>
        <el-checkbox v-model="recvOption.subpackage">分包显示</el-checkbox>
      </div>
      <div style="margin-top: 10px;">
        <span>端口号</span>
        <el-select
          v-model="channel"
          placeholder="请选择端口号"
          size="mini"
          value-key="name"
          style="margin-left: 12px; width: 280px;"
          :disabled="linkInfo.linked || linkInfo.listened"
        >
          <el-option
            v-for="(item, index) in channelList"
            :key="index"
            :label="item.name"
            :value="item"
          ></el-option>
        </el-select>
      </div>
      <div v-if="channel.type === 'serial'">
        <div style="margin-top: 2px;">
          <el-button
            :type="linkInfo.linked ? 'success' : 'primary'"
            size="mini"
            @click="onClickOpen"
          >{{ linkInfo.linked ? '关闭串口' : '打开串口' }}</el-button>
          <span style="font-size: 12px; margin-left: 10px;">波特率</span>
          <el-select
            v-model="serialportOption.baudRate"
            placeholder="波特率"
            filterable
            size="mini"
            style="width: 100px;"
            :disabled="linkInfo.linked"
          >
            <el-option
              v-for="(item, index) in baudRate"
              :key="index"
              :value="item"
            ></el-option>
          </el-select>
          <span style="font-size: 12px; margin-left: 10px;">校验</span>
          <el-select
            v-model="serialportOption.parity"
            placeholder="校验"
            filterable
            size="mini"
            style="width: 80px;"
            :disabled="linkInfo.linked"
          >
            <el-option
              v-for="(item, index) in parity"
              :key="index"
              :value="item"
            ></el-option>
          </el-select>
        </div>
        <div>
          <span style="font-size: 12px; margin-left: 90px;">数据位</span>
          <el-select
            v-model="serialportOption.dataBits"
            placeholder="数据位"
            filterable
            size="mini"
            style="width: 80px;"
            :disabled="linkInfo.linked"
          >
            <el-option
              v-for="(item, index) in dataBits"
              :key="index"
              :value="item"
            ></el-option>
          </el-select>
          <span style="font-size: 12px; margin-left: 18px;">停止位</span>
          <el-select
            v-model="serialportOption.stopBits"
            placeholder="停止位"
            filterable
            size="mini"
            style="width: 80px;"
            :disabled="linkInfo.linked"
          >
            <el-option
              v-for="(item, index) in stopBits"
              :key="index"
              :value="item"
            ></el-option>
          </el-select>
        </div>
      </div>
      <div v-else>
        <div>
          <span>远程</span>
          <el-input
            v-model="remote.host"
            placeholder="请输入IP或域名"
            size="mini"
            style="width: 130px; margin-left: 12px;"
            :disabled="channel.type === 'server' || linkInfo.linked || linkInfo.listened"
          ></el-input>
          <el-input-number
            v-model="remote.port"
            placeholder="请输入端口"
            size="mini"
            controls-position="right"
            :min="1"
            :max="65535"
            style="width: 110px;"
            :disabled="channel.type === 'server' || linkInfo.linked || linkInfo.listened"
          ></el-input-number>
          <el-button
            type="primary"
            size="mini"
            @click="onClickConnect"
            :disabled="channel.type === 'server'"
          >{{ !linkInfo.listened && linkInfo.linked ? "断开" : "连接" }}</el-button>
        </div>
        <div style="margin-bottom: 10px;">
          <span>本地</span>
          <el-select
            v-model="local.host"
            placeholder="请选择IP"
            size="mini"
            style="width: 130px; margin-left: 12px;"
            :disabled="channel.type === 'client' || linkInfo.linked || linkInfo.listened"
          >
            <el-option
              v-for="item in localAddress"
              :key="item"
              :label="item"
              :value="item"
            ></el-option>
          </el-select>
          <el-input-number
            v-model="local.port"
            placeholder="请输入端口"
            size="mini"
            controls-position="right"
            :min="1"
            :max="65535"
            style="width: 110px;"
            :disabled="channel.type === 'client' || linkInfo.linked || linkInfo.listened"
          ></el-input-number>
          <el-button
            type="primary"
            size="mini"
            :disabled="channel.type === 'client'"
            @click="onClickListen"
          >{{ linkInfo.listened ? "断开" : "监听" }}</el-button>
        </div>
      </div>
      <div class="function-area-bottom">
        <el-button
          type="danger"
          size="mini"
          @click="onClickSendClear"
        >清除发送窗口</el-button>
        <div>
          <el-checkbox v-model="sendOption.hex">HEX发送</el-checkbox>
        </div>
        <div>
          <el-checkbox v-model="sendOption.autoSend" :disabled="!linkInfo.linked">定时发送</el-checkbox>
          <el-input-number
            v-model="sendOption.interval"
            size="mini"
            controls-position="right"
            style="margin-left: 12px;"
            :disabled="sendOption.autoSend"
          ></el-input-number>
          <span style="font-size: 12px;">ms/次</span>
        </div>
        <el-button
          type="primary"
          class="send-button"
          @click="onClickSend"
          :disabled="!linkInfo.linked"
        >发送</el-button>
      </div>
    </div>
    <!-- 发送区域 -->
    <div class="send-area">
      <el-input
        type="textarea"
        v-model="sendOption.text"
        style="height: 100%;"
        resize="none"
      ></el-input>
    </div>
    <!-- 状态栏 -->
    <div class="status-bar">
      <div class="left">
        <span>R: {{recvOption.length}}</span>
        <span style="margin-left: 15px;">S: {{sendOption.length}}</span>
      </div>
      <div class="center">
        <span>{{ linkInfo.message }}</span>
      </div>
      <div class="right">
        <span>行 0，列 0</span>
      </div>
    </div>
  </div>
</template>

<script>
const os = require('os');
const net = require('net');
const dgram = require('dgram');
const moment = require('moment');
const SerialPort = require('serialport');
const InterByteTimeout = require('@serialport/parser-inter-byte-timeout');

export default {
  name: 'home',
  data() {
    return {
      recvOption: {
        text: '', hex: true, timestamp: true, subpackage: true, length: 0,
      },
      sendOption: {
        text: '', hex: true, length: 0, autoSend: false, interval: 1000,
      },
      channel: { name: 'TCPClient', type: 'client' },
      channelList: [
        { name: 'TCPClient', type: 'client' },
        { name: 'TCPServer', type: 'server' },
        { name: 'UDPClient', type: 'client' },
        { name: 'UDPServer', type: 'server' },
      ],
      remote: { host: '127.0.0.1', port: 8899 },
      local: { host: '127.0.0.1', port: 8899 },
      localAddress: [],
      socket: null,
      server: null,
      linkInfo: {
        linked: false,
        listened: false,
        message: '',
      },
      recvWindow: null,
      job: null,
      serialportOption: {
        baudRate: 9600,
        dataBits: 8,
        stopBits: 1,
        parity: 'none',
      },
      port: null,
      baudRate: [
        110, 300, 1200, 2400, 4800, 9600, 14400, 19200, 38400, 57600, 115200, 128000,
      ],
      dataBits: [8, 7, 6, 5],
      stopBits: [1, 2],
      parity: ['none', 'even', 'mark', 'odd', 'space'],
    };
  },

  methods: {
    buf2hex(buffer) {
      return buffer.toString('hex').toUpperCase().replace(/\s/g, '').replace(/(.{2})/g, '$1 ');
    },

    showRecv(data) {
      let result = data;
      if (this.recvOption.hex) {
        result = this.buf2hex(result);
      }
      if (this.recvOption.subpackage) {
        result = `${result}\r\n`;
      }
      if (this.recvOption.timestamp) {
        result = `[${moment().format('YYYY-MM-DD HH:mm:ss.SSS')}] <- ${result}`;
      }
      this.recvOption.text += result;
      this.recvOption.length += data.length;
      this.recvWindow.scrollTop = this.recvWindow.scrollHeight;
    },

    showSend(data) {
      let result = data;
      if (this.recvOption.hex) {
        result = this.buf2hex(result);
      }
      if (this.recvOption.subpackage) {
        result = `${result}\r\n`;
      }
      if (this.recvOption.timestamp) {
        result = `[${moment().format('YYYY-MM-DD HH:mm:ss.SSS')}] -> ${result}`;
      }
      this.recvOption.text += result;
      this.recvWindow.scrollTop = this.recvWindow.scrollHeight;
    },

    getLocalAddress() {
      const ret = [];
      const interfaces = os.networkInterfaces();
      Object.values(interfaces).forEach((iface) => {
        for (let i = 0, len = iface.length; i < len; i += 1) {
          const info = iface[i];
          if (info.family === 'IPv4') {
            ret.push(info.address);
            break;
          }
        }
      });
      return ret;
    },

    doConnect(host, port) {
      const socket = net.connect(port, host);
      socket.on('connect', () => {
        this.linkInfo.linked = true;
        this.linkInfo.message = `${socket.localAddress}:${socket.localPort} >>> ${host}:${port}`;
      });
      socket.on('data', (data) => {
        this.showRecv(data);
      });
      socket.on('close', () => {
        this.linkInfo.linked = false;
        this.linkInfo.message = '连接已断开';
      });
      socket.on('error', (err) => {
        this.$notify.error({
          title: '连接错误',
          message: err,
        });
      });
      this.socket = socket;
    },

    doConnectUDP(host, port) {
      const socket = dgram.createSocket('udp4');
      socket.on('connect', () => {
        this.linkInfo.linked = true;
        const address = socket.address();
        this.linkInfo.message = `${address.address}:${address.port} >>> ${host}:${port}`;
      });
      socket.on('message', (msg) => {
        this.showRecv(msg);
      });
      socket.connect(port, host);
      this.socket = socket;
    },

    onClickConnect() {
      if (this.channel.name === 'TCPClient') {
        if (!this.linkInfo.linked) {
          this.doConnect(this.remote.host, this.remote.port);
        } else {
          this.socket.end();
        }
      } else if (this.channel.name === 'UDPClient') {
        if (!this.linkInfo.linked) {
          this.doConnectUDP(this.remote.host, this.remote.port);
        } else {
          this.socket.disconnect();
          this.linkInfo.linked = false;
          this.linkInfo.message = '已断开';
        }
      }
    },

    doListen(host, port) {
      const server = net.createServer();
      server.maxConnections = 1;
      server.on('connection', (socket) => {
        this.linkInfo.linked = true;
        this.linkInfo.message = `${host}:${port} <<< ${socket.remoteAddress}:${socket.remotePort}`;
        socket.on('data', (data) => {
          this.showRecv(data);
        });
        socket.on('close', () => {
          this.linkInfo.linked = false;
          if (this.linkInfo.listened) {
            this.linkInfo.message = `TCP listen ${host}:${port}`;
          }
        });
        this.socket = socket;
      });
      server.on('close', () => {
        this.linkInfo.linked = false;
        this.linkInfo.listened = false;
        this.linkInfo.message = '停止监听';
      });
      server.on('listening', () => {
        this.linkInfo.listened = true;
        this.linkInfo.message = `TCP listen ${host}:${port}`;
      });
      server.on('error', (err) => {
        this.$notify.error({
          title: '监听错误',
          message: err,
        });
      });
      server.listen(port, host);
      this.server = server;
    },

    doListenUDP(host, port) {
      const server = dgram.createSocket('udp4');
      server.on('listening', () => {
        this.linkInfo.listened = true;
        this.linkInfo.message = `UDP listen ${host}:${port}`;
      });
      server.on('close', () => {
        this.linkInfo.listened = false;
        this.linkInfo.message = '停止监听';
      });
      server.on('error', (err) => {
        this.$notify.error({
          title: '监听错误',
          message: err,
        });
        server.close();
      });
      server.on('message', (msg, rinfo) => {
        this.showRecv(msg);
        this.linkInfo.message = `${host}:${port} <<< ${rinfo.address}:${rinfo.port}`;
        this.linkInfo.linked = true;
        this.remote.host = rinfo.address;
        this.remote.port = rinfo.port;
      });
      server.bind(port, host);
      this.server = server;
    },

    onClickListen() {
      if (!this.linkInfo.listened) {
        if (this.channel.name === 'TCPServer') {
          this.doListen(this.local.host, this.local.port);
        } else if (this.channel.name === 'UDPServer') {
          this.doListenUDP(this.local.host, this.local.port);
        }
      } else {
        this.server.close();
        if (this.socket) {
          this.socket.destroy();
        }
      }
    },

    onClickSendClear() {
      this.sendOption.text = '';
      this.sendOption.length = 0;
    },

    onClickRecvClear() {
      this.recvOption.text = '';
      this.recvOption.length = 0;
    },

    preSend() {
      let sendBuf;
      if (this.sendOption.hex) {
        sendBuf = Buffer.from(this.sendOption.text.replace(/\s/g, ''), 'hex');
      } else {
        sendBuf = Buffer.from(this.sendOption.text);
      }
      this.showSend(sendBuf);
      this.sendOption.length += sendBuf.length;
      return sendBuf;
    },

    onClickSend() {
      switch (this.channel.type) {
        case 'client':
          if (this.channel.name === 'TCPClient') {
            this.socket.write(this.preSend());
          } else if (this.channel.name === 'UDPClient') {
            this.socket.send(this.preSend());
          }
          break;
        case 'server':
          if (this.channel.name === 'TCPServer') {
            this.socket.write(this.preSend());
          } else if (this.channel.name === 'UDPServer') {
            this.server.send(this.preSend(), this.remote.port, this.remote.host);
          }
          break;
        case 'serial':
          this.port.write(this.preSend());
          break;
        default:
          break;
      }
    },

    updateSerialport() {
      SerialPort.list().then(
        (ports) => {
          ports.forEach((port) => {
            this.channelList.unshift({ name: port.path, type: 'serial' });
          });
          this.channel = this.channelList[0];
        },
      );
    },

    onClickOpen() {
      if (this.linkInfo.linked) {
        this.port.close((err) => {
          if (err) {
            this.$message.error(`${this.channel.name}关闭失败`);
          } else {
            this.linkInfo.linked = false;
          }
        });
        this.port = null;
      } else {
        const port = new SerialPort(this.channel.name, {
          autoOpen: false,
          ...this.serialportOption,
        });
        port.open((err) => {
          if (err) {
            this.$message.error(`${this.channel.name}打开失败`);
          } else {
            this.linkInfo.linked = true;
          }
        });
        const parser = port.pipe(new InterByteTimeout({ interval: 30 }));
        parser.on('data', (data) => {
          this.showRecv(data);
        });
        this.port = port;
      }
    },
  },

  mounted() {
    this.localAddress = this.getLocalAddress();
    this.recvWindow = document.getElementById('recv-window');
    this.updateSerialport();
  },

  computed: {
    autoSend() {
      return this.sendOption.autoSend;
    },
    linked() {
      return this.linkInfo.linked;
    },
  },

  watch: {
    autoSend() {
      if (this.autoSend) {
        let task;
        switch (this.channel.type) {
          case 'client':
            if (this.channel.name === 'TCPClient') {
              task = () => this.socket.write(this.preSend());
            } else if (this.channel.name === 'UDPClient') {
              task = () => this.socket.send(this.preSend());
            }
            break;
          case 'server':
            if (this.channel.name === 'TCPServer') {
              task = () => this.socket.write(this.preSend());
            } else if (this.channel.name === 'UDPServer') {
              task = () => this.server.send(this.preSend(), this.remote.port, this.remote.host);
            }
            break;
          case 'serial':
            task = () => this.port.write(this.preSend());
            break;
          default:
            break;
        }
        this.job = setInterval(task, this.sendOption.interval);
      } else {
        clearInterval(this.job);
      }
    },
    linked() {
      if (!this.linked && this.job) {
        this.sendOption.autoSend = false;
        clearInterval(this.job);
      }
    },
  },
};
</script>

<style lang="scss" scoped>
.container {
  height: 100%;
  position: relative;
}

.el-textarea::v-deep .el-textarea__inner {
  height: 100% !important;
  border-radius: 0;
  font-family: Monaco;
  color: black;
  font-size: 11px;
  padding: 2px;
}
.el-checkbox::v-deep .el-checkbox__label {
  font-size: 13px;
}

.recv-area {
  position: absolute;
  width: 100%;
  top: 0;
  bottom: 255px;
}

.function-area {
  width: 350px;
  height: 230px;
  position: absolute;
  left: 0;
  bottom: 18px;
  padding: 5px;
}

.function-area-bottom {
  position: absolute;
  bottom: 8px;
  right: 0;
  left: 5px;
}

.send-button {
  position: absolute;
  right: 14px;
  bottom: 10px;
  width: 50px;
  height: 56px;
}
.send-button::v-deep span {
  writing-mode: vertical-lr;
}

.send-area {
  position: absolute;
  left: 360px;
  right: 0;
  bottom: 18px;
  height: 236px;
}

.status-bar {
  position: absolute;
  background: rgb(0, 122, 204);
  width: 100%;
  height: 18px;
  bottom: 0;
  font-size: 14px;
  color: white;
  line-height: 18px;
  .left {
    display: inline;
    position: absolute;
    left: 5%;
  }
  .center {
    display: inline;
    position: absolute;
    left: 30%;
  }
  .right {
    display: inline;
    position: absolute;
    right: 10%;
  }
}
</style>
