<template>
  <div
    class="journey"
  >
    <p class="journey-title">
      行程:
      <span>{{journeyInfo.startPlace}}{{(journeyInfo.endPlace ? ' - ' : '') +journeyInfo.endPlace}} <button class="button-delete-journey button-delete" @click="deleteJourney">删除行程</button> </span>

    </p>
    <div class="journey-info">
      <label class="journey-info-item"><span class="name">起始日期</span><input type="date" v-model="journeyInfo.startTime"/></label>
      <label class="journey-info-item"><span class="name">终止日期</span><input type="date" v-model="journeyInfo.endTime"/></label>
      <label class="journey-info-item"><span class="name">起始地点</span><input type="text" v-model="journeyInfo.startPlace"/></label>
      <label class="journey-info-item"><span class="name">终止地点</span><input type="text" v-model="journeyInfo.endPlace"/></label>
    </div>
    <div class="reimburse-block">
      <button class="button-add-reimburse " @click="addReimburse">增加发票 </button>
      <div
        v-for="(item, index) in reimbursement"
        :key="index"
        class="reimburse-item"
      >
        <label>
          <span>发票类型</span>
          <select v-model="item.type">
            <option
              v-for="r in reimbursementType"
              :key="r.type"
              :value="r.type"
            >{{r.text}}</option>
          </select>
        </label>
        <label>
          <span>费用</span>
          <input type="text" v-model="item.value"/>
        </label>
        <button class="button-delete-reimburse button-delete" @click="deleteReimburse(index)">删除</button>
      </div>
    </div>
  </div>
</template>
<script>
export default {
  name: '',
  props: {
    counterKey: {},
    reimbursementType: {
      type: Array,
      default: () => []
    }
  },
  components: {},
  mixins: [],
  data() {
    return {
      journeyInfo: {
        startTime: '',
        endTime: '',
        startPlace: '',
        endPlace: ''
      },
      reimbursement: [],
      tableTitle: []
    };
  },
  created() {},
  mounted() {},
  beforeDestroy() {},
  filters: {},
  watch: {
    journeyInfo: {
      deep: true,
      handler() {
        this.emit();
      }
    },
    reimbursement: {
      deep: true,
      handler() {
        this.emit();
      }
    },
  },
  computed: {},
  methods: {
    addReimburse() {
      this.reimbursement.push(
        {
          type: '',
          value: ''
        }
      );
    },
    deleteReimburse(index) {
      this.reimbursement.splice(index, 1);
    },
    emit() {
      this.$nextTick(()=>{
        this.$emit('input', {
          ...this.journeyInfo,
          reimbursement: this.reimbursement,
          key: this.counterKey
        })
      })
    },
    deleteJourney() {
      this.$emit('delete');
    }
  }
};
</script>
<style lang="scss" scoped>
.button-delete{
  background: #fff;
  height: 30px;
  border: 2px solid #ddd;
  color: #716b6b;
  font-weight: bold;
  font-size: 14px;
  padding: 0 10px;
  outline: none;
  -webkit-transition: all 0.1s;
  transition: all 0.1s;
  cursor: pointer;
  &:hover{
    background-color: rgba(107, 106, 109, 0.16);
  }
}
.journey{
  padding: 1px 10px 14px;
  label{
    color: #505050;
    margin-right: 30px;
  }
  input[type='text'], input[type='date'], select{
    margin-left: 6px;
    border: none;
    border-bottom: 1px solid #9c9c9c;
    background-color: transparent;
    font-size: 18px;
    color: #000;
    outline: none;
    padding-left: 10px;
    height: 32px;
    line-height: 32px;
  }
  input[type='text']{
    width: 100px;
  }
  .journey-title{
    position: relative;
    font-size: 24px;
    line-height: 1.08;
    font-weight: 400;
    color: #333;
    padding-bottom: 11px;
    padding-left: 10px;
    border-bottom: 1px solid #d6d6d6;
    span{
      margin-left: 6px;
    }
  }
  .button-delete-journey{
    position: relative;
    top: -3px;
    left: 12px;
  }
}
.journey-info{
  padding-left: 10px;
  .journey-info-item{
    display: inline-block;
    .name{
      margin-right: 5px;
    }
  }

}
.reimburse-block{
  padding-left: 10px;
  margin-top: 20px;
  .reimburse-item{
    margin-bottom: 6px;
    transition: all 0.1s;
    .button-delete-reimburse{
      margin-left: 10px;
    }
  }
  .button-add-reimburse{
    position: relative;
    height: 30px;
    margin-bottom: 5px;
    background: #fff;
    border: 2px solid #000;
    font-weight: bold;
    font-size: 14px;
    padding: 0 10px;
    outline: none;
    -webkit-transition: all 0.1s;
    transition: all 0.1s;
    cursor: pointer;
    &:hover{
      background-color: rgba(107, 106, 109, 0.16);
    }
  }

}
</style>
