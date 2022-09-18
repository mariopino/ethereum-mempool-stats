<template>
  <b-container fluid>
    <b-row class="mb-2">
      <b-col cols="12">
        <h1>mempool transactions</h1>
        <div class="table-responsive">
          <b-table striped hover :fields="fields" :items="lastTxs" />
        </div>
      </b-col>
    </b-row>
  </b-container>
</template>

<script>
import Web3 from 'web3';
import axios from 'axios';
import { ethers } from 'ethers';

const ethWs = 'wss://solitary-thrilling-sanctuary.discover.quiknode.pro/7fd63709abe3764c33ec3d0c6e689e3f2a80c0f6/';
const options = {
  timeout: 30000,
  clientConfig: {
    maxReceivedFrameSize: 100000000,
    maxReceivedMessageSize: 100000000,
  },
  reconnect: {
    auto: true,
    delay: 5000,
    maxAttempts: 15,
    onTimeout: false,
  },
};

export default {
  name: 'IndexPage',
  data() {
    return {
      lastTxs: [],
      size: 20,
      fields: [
      {
          key: 'timestamp',
          label: 'Timestamp',
          sortable: false,
        },
        {
          key: 'from',
          label: 'From',
          sortable: false,
        },
        {
          key: 'to',
          label: 'To',
          sortable: false,
        },
        {
          key: 'maxFee',
          label: 'Max fee',
          sortable: false,
        },
        {
          key: 'gas',
          label: 'Gas',
          sortable: false,
        },
        {
          key: 'gasPrice',
          label: 'Gas price',
          sortable: false,
        },
        {
          key: 'maxFeePerGas',
          label: 'Max fee per gas',
          sortable: false,
        },
        {
          key: 'maxPriorityFeePerGas',
          label: 'Max priority fee per gas',
          sortable: false,
        },



      ]
    }
  },
  async created() {
    await this.getEthGas();
    
    const web3 = new Web3(new Web3.providers.WebsocketProvider(ethWs, options));
    const subscription = web3.eth.subscribe("pendingTransactions", (err, res) => {
      if (err) console.error(err);
    });

    subscription.on("data", (txHash) => {
      setTimeout(async () => {
        try {
          const tx = await web3.eth.getTransaction(txHash);
          if (tx !== null) {
            tx.timestamp = new Date().getTime();

            tx.maxFee = `${ethers.utils.formatUnits(
              ethers.BigNumber.from(tx.gas.toString()).mul(ethers.BigNumber.from(tx.gasPrice.toString())),
              'ether'
            )} ETH`;

            console.log(tx);
            this.lastTxs.unshift(tx)
            if (this.lastTxs.length > this.size) {
              this.lastTxs.pop();
            }
          }
        } catch (err) {
          console.error(err);
        }
      });
    });

  },
  methods: {
    async getEthGas() {
      const response = await axios.get(
      'https://ethgasstation.info/api/ethgasAPI.json?',
    );
    this.ethGas = response.data;
    console.log(this.ethGas)
    }
  }
}
</script>
