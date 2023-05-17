<template>
  <div class="home">
    <div class="button-list">
      <div class="button-item" @click="createBot">+ Bot</div>
      <div class="button-item" @click="removeBot">- Bot</div>
      <div class="button-item" @click="createOrder(false)">+ Normal Order</div>
      <div class="button-item" @click="createOrder(true)">+ VIP Order</div>
    </div>
    <div class="bot-list" v-if="botList.length > 0">
      <div class="bot-item" v-for="(item, index) in botList" :key="index">
        Bot {{ index + 1 }}
        <div v-if="item.orderId != ''">
          <div>Order <span class="order-id" :class="item.isVip ? 'vip' : 'normal'">{{ item.orderId }}</span></div>
          <div>{{ item.time }} s</div>
        </div>
        <div v-else>Idle</div>
      </div>
    </div>
    <div class="area">
      <div class="area-item">
        <div class="area-title">Pending Area</div>
        <div v-if="pendingList.length > 0" class="area-list">
          <div class="area-order" :class="item.isVip ? 'vip' : 'normal'" v-for="(item, index) in showPendingList" :key="index">
            #{{ item.id }}
          </div>
        </div>
      </div>
      <div class="area-item">
        <div class="area-title">Complete Area</div>
        <div v-if="completeList.length > 0" class="area-list">
          <div class="area-order" :class="item.isVip ? 'vip' : 'normal'" v-for="(item, index) in completeList" :key="index">
            #{{ item.id }}
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
export default {
  name: 'Home',
  data() {
    return {
      isLoading: false,
      index: 1,
      botIndex: 1,
      botList: [],
      pendingList: [],
      completeList: [],
      intervalList: [],
    }
  },
  methods: {
    createOrder(isVip) {
      if (!this.isLoading) {
        this.isLoading = true;
        var data = {
          id: (isVip ? 'V' : 'N') + this.index,
          isVip: isVip,
          isPickUp: false,
        };

        if (isVip) {
          var idx = this.pendingList.map(item => item.isVip).lastIndexOf(true);
          if (idx == -1) {
            this.pendingList.unshift(data);
          } else {
            this.pendingList.splice(idx + 1, 0, data);
          }
        } else {
          this.pendingList.push(data);
        }

        this.index++;
        this.handleOrder();
        this.isLoading = false;
      }
    },
    createBot() {
      if (!this.isLoading) {
        this.isLoading = true;
        var data = {
          id: this.botIndex,
          orderId: '',
          isVip: null,
          time: 10,
        };
  
        this.botList.push(data);
        this.botIndex++;
        this.handleOrder();
        this.isLoading = false;
      }
    },
    removeBot() {
      if (!this.isLoading && this.botList.length > 0) {
        this.isLoading = true;
        var lastBot = this.botList[this.botList.length - 1];
        if (lastBot['orderId'] != '') {
          var idx = this.pendingList.map(item => item.id).indexOf(lastBot['orderId']);
          if (this.pendingList[idx]['isPickUp'] == true) {
            clearInterval(this.intervalList[lastBot['id'] - 1]);
            this.pendingList[idx]['isPickUp'] = false;
            lastBot['orderId'] = '';
            lastBot['isVip'] = null;
            lastBot['time'] = 10;
          }
        }
        this.botList.pop();
        this.handleOrder();
        this.isLoading = false;
      }
    },
    handleOrder() {
      if (this.pendingList.length > 0 && this.botList.length > 0) {
        var idx = this.botList.map(item => item.orderId).indexOf('');
        var pendingIdx = this.pendingList.map(item => item.isPickUp).indexOf(false);
        if (idx != -1 && pendingIdx != -1) {
          this.botList[idx]['orderId'] = this.pendingList[pendingIdx]['id'];
          this.botList[idx]['isVip'] = this.pendingList[pendingIdx]['isVip'];
          this.pendingList[pendingIdx]['isPickUp'] = true;
          this.setTimer(this.botList[idx]);
        }
      }
    },
    setTimer(bot) {
      this.intervalList[bot['id'] - 1] = setInterval(() => {
        bot['time']--;
        if (bot['time'] <= 0) {
          clearInterval(this.intervalList[bot['id'] - 1]);
          this.completeList.push({ id: bot['orderId'], isVip: bot['isVip'] });
          bot['orderId'] = '';
          bot['isVip'] = null;
          bot['time'] = 10;
          this.handleOrder();
        }
      }, 1000);
    },
  },
  computed: {
    showPendingList() {
      return this.pendingList.filter(i => i.isPickUp == false); 
    }
  },
}
</script>
<style lang="scss" scoped>
  .home{
    position: relative;

    .vip{
      background-color: rgba(255, 0, 0, 0.2);
    }

    .normal{
      background-color: rgba(0, 255, 0, 0.2);
    }

    .button-list{
      display: flex;
      flex-wrap: wrap;
      justify-content: center;
      gap: 20px;

      .button-item{
        padding: 10px 20px;
        border: 1px solid black;
        border-radius: 4px;
        cursor: pointer;
      }
    }

    .bot-list{
      padding-top: 20px;
      display: flex;
      flex-wrap: wrap;
      justify-content: center;
      gap: 20px;

      .bot-item{
        padding: 10px 20px;
        border: 1px solid red;
        border-radius: 4px;
        display: flex;
        flex-direction: column;
        row-gap: 5px;

        .order-id{
          padding: 3px 10px;
          border-radius: 4px;
        }
      }
    }

    .area{
      padding-top: 20px;
      display: grid;
      grid-template-columns: repeat(2, 1fr);
      column-gap: 10px;

      .area-item{
        padding: 10px;
        border: 1px solid black;
        border-radius: 4px;

        .area-title{
          font-weight: bold;
        }

        .area-list{
          display: flex;
          flex-wrap: wrap;
          gap: 10px;
          padding-top: 20px;

          .area-order{
            padding: 10px 15px;
            border-radius: 4px;
          }
        }
      }
    }
  }
</style>