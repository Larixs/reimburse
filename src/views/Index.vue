<template>
  <div class="reimburse">
    <div>
      <!--<h2>输入发票信息</h2>-->
      <button
        class="button-add-journey button-add"
        @click="addJourney"
      >增加行程</button>
      <div class="journey-block">
        <div
          v-for="(item, index) in allJourney"
          :key="item.key"
        >
          <Journey
            @input="(val)=>{journeyChange(val,index)}"
            class="journey-item"
            @delete="deleteJourney(index)"
            :counterKey="item.key"
            :reimbursementType="reimbursementType"
          />
        </div>

      </div>

    </div>
    <div v-if="allJourney.length">
      <button class="button-add button-start-counting" @click="()=> {calculate('paper')}">开始计算贴发票内容</button>
      <button class="button-add button-start-counting" @click="()=> {calculate('system')}">开始计算报销系统内容</button>
      <table class="calculate-result">
        <thead v-if="calculateResult.length">
          <th></th>
          <th v-for="(item,index) in tableHead" :key="index">{{item}}</th>
        </thead>
        <tbody>
          <tr v-for="(row, rowIndex) in calculateResult" :key="rowIndex">
            <td></td>
            <td v-for="(cell, cellIndex) in row" :key="cellIndex">
              {{cell|isZero}}
            </td>
          </tr>
        </tbody>
        <tfoot>
          <tr v-if="calculateResult.length">
            <td>总计</td>
            <td
              v-for="(colResult, index) in colResults"
              :key="index"
            >{{colResult|isZero}}</td>
          </tr>
        </tfoot>
      </table>
      <div class="total" v-if="allInTotal">
        总计： {{allInTotal}} <br>
        <p>
          <span>大写金额:</span> {{nzhNumber}}
        </p>
      </div>
    </div>
  </div>
</template>
<script>
import Journey from "../components/index/Journey.vue";
import moment from 'moment';
import nzh from 'nzh';
export default {
  name: '',
  props: {},
  components: { Journey },
  mixins: [],
  data() {
    return {
      allJourney: [],
      journeyCounter: 0,
      calculateResult: [],
      reimbursementType: [
        {
          type: 'air',
          text: '机票费'
        },
        {
          type: 'train',
          text: '车船费'
        },
        {
          type: 'city-traffic',
          text: '市内交通费'
        },
        {
          type: 'hotel',
          text: '住宿费'
        }
      ],
      tableHead: [],
      allInTotal: 0,
      nzhNumber: '',
      colResults: []
    };
  },
  created() {},
  mounted() {},
  beforeDestroy() {},
  filters: {
    isZero(val){
      if(parseFloat(val) === 0){
        return '/'
      }else{
        return val;
      }
    }
  },
  watch: {
    nzhNumber() {
      this.nzhNumberHtml = '';
    }
  },
  computed: {},
  methods: {
    addJourney() {
      this.journeyCounter= this.journeyCounter+1;
      this.allJourney.push({
        key: this.journeyCounter
      })
    },
    journeyChange(val, journeyIndex){
      this.allJourney.splice(journeyIndex, 1, val);
    },
    deleteJourney(index) {
      this.allJourney.splice(index, 1);
    },
    calculate(type='paper') {
      if(type === 'paper'){
        this.tableHead = ['日期', '起讫地点', '天数', ...this.reimbursementType.map(i=>i.text), '出差补助', '小计']
      } else if (type === 'system') {
        this.tableHead = ['日期', '起讫地点', '天数', '交通费', '住宿费', '员工福利', '小计'];
      } else {
        alert('error in calculate');
        console.log('error in calculate', type);
        return;
      }
      const sortedJourney = [...this.allJourney].sort((next, last)=> new Date(next.startTime).getTime() - new Date(last.startTime));
      const preResult = sortedJourney.map(i => this.calculateSingleJourney(i, type));
      this.calculateResult = preResult.map(i=>[i.startTime, i.place, i.duration, ...i.cost, i.allowance, i.all]);
      this.colResults = this.calculateColResults(this.calculateResult);
      this.allInTotal = this.calculateResult.reduce((last, next) => last + next[next.length - 1], 0);
      this.nzhNumber = nzh.cn.toMoney(this.allInTotal, {complete:true, outSymbol:false});
    },
    calculateSingleJourney(journeyInfo, type='paper') {
      const place = `${journeyInfo.startPlace}-${journeyInfo.endPlace}`;
      const duration = this.getDuration(journeyInfo.startTime, journeyInfo.endTime);
      const allowance = this.getAllowance(duration);
      let invoice = this.getInvoice(journeyInfo.reimbursement, type);
      const all = invoice.reduce((last, next) => last + next, allowance);
      return {
        startTime: journeyInfo.startTime,
        place,
        duration,
        cost: invoice,
        allowance,
        all
      };
    },
    calculateColResults(res) {
      if (res.length > 0 && res[0].length) {
        const r = new Array(res[0].length).fill(0);
        res.map(item=>{
          for(let j = 0; j < item.length; j++){
            const f = +(item[j]);
            if(!isNaN(f)){
              r[j] += f;
            }
          }
        });
        return r;
      } else {
        return [];
      }
    },
    getDuration(startTime, endTime) {
      const _startTime = moment(startTime, 'YYYY-MM-DD');
      const _endTime = moment(endTime, 'YYYY-MM-DD');
      return _endTime.diff(_startTime, 'days') + 1;
    },
    getAllowance(duration) {
      return duration * 30;
    },
    getInvoice(invoices, type='paper') {
      const result = {};
      for(let i = 0; i< invoices.length; i++){
        const invoice = invoices[i];
        result[invoice.type] = (result[invoice.type] || 0) + parseFloat(invoice.value * 100 || 0) // 浮点计算不准很蓝瘦
      }
      if (type === 'system') {
        const traffic = ((result.air || 0) + (result.train || 0) + (result['city-traffic'] || 0))/100;
        const hotel = (result.hotel || 0) / 100;
        return [traffic, hotel];
      } else {
        return this.reimbursementType.map(i => (result[i.type] || 0) / 100);
      }
    }
  }
};
</script>
<style lang="scss" scoped>
.reimburse{
  padding: 20px;
  font-family: 'HanHei SC', 'PingFang SC', 'Helvetica Neue', 'Helvetica', 'STHeitiSC-Light', 'Arial', sans-serif;
  h2{
    font-size: 40px;
    color: #333;
    font-weight: normal;
    line-height: 1.0833;
  }
  .button-add{
    background: #fff;
    height: 44px;
    border: 2px solid black;
    font-weight: bold;
    font-size: 20px;
    padding: 0 20px;
    outline: none;
    transition: all 0.1s;
    cursor: pointer;
    &:hover{
      background-color: rgba(107, 106, 109, 0.16);
    }
  }
  .journey-block{
     padding: 20px 0;
     .journey-item{
       margin-bottom: 40px;
       background-color: #fafafa;
     }
   }
  .button-start-counting{
    margin-right: 16px;
  }
  table.calculate-result{
    margin: 30px 0;
    text-align: center;
    border-collapse: collapse;
    thead{
      background-color: #eaeaea;
      th{
        min-width: 110px;
        line-height: 40px;
      }
    }
    tbody{
      tr{
        background-color: #fafafa;
        line-height: 30px;
        transition: all 0.1s;
        &:hover{
          background-color: #dadada;
        }
        &:nth-child(even){
          background-color: #f0f0f0;
        }
      }
    }
    tfoot{
      background-color: #fafafa;
      border-top: 1px solid #e0e0e0;
    }
  }
}

</style>
