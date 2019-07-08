<template>
  <div>
    <q-page class="flex-center">
      <div class="column" style="height: 150px">
        <div class="col">
          <q-btn label="추가" color="primary" @click="confirm = true" />
          / {{priceTotal}}
        </div>
        <div class="col">
          <q-table title="다음과 같이 견적합니다." :data="data" :columns="columns"></q-table>
        </div>
      </div>
    </q-page>
    <q-dialog v-model="confirm" persistent>
      <q-card>
        <q-card-section class="row items-center">
          <div class="row items-start">
            <q-select v-model="elementType" :options="options" label="소재 타입" style="width: 300px" />
          </div>
          <div class="row items-start">
            <q-input v-model.number="t" type="number" label="T" style="width: 100px" />
            <q-input v-model.number="w" type="number" label="W" style="width: 100px" />
            <q-input v-model.number="l" type="number" label="L" style="width: 100px" />
            <q-input v-model.number="quantity" type="number" label="수량" style="width: 100px" />
            <q-input v-model.number="unitPrice" type="number" label="단가" style="width: 100px" />
          </div>
          <div class="row items-start">
            <q-checkbox v-model="showWeight" label="소재 무게 표시" />
            <q-checkbox v-model="showBottomProcess" label="바닥면 정삭 표시" />
          </div>
          <div class="row items-start">
            <q-select
              v-model="additionalElementType"
              :options="additionalOptions.map(option => option.name)"
              label="추가 입력"
              style="width: 300px"
            />
            <q-input
              v-model.number="subItemQuantity"
              type="number"
              label="수량"
              style="width: 100px"
            />
            <q-input
              v-model.number="subItemUnitPrice"
              type="number"
              label="단가"
              style="width: 100px"
            />
            <q-btn flat label="추가" @click="addSubItem" color="primary" />
          </div>
          <ul>
            <li
              v-for="(item, index) in additionalData"
              :key="index"
            >{{item.name}} / {{item.quantity}} / {{item.unitPrice}}</li>
          </ul>
        </q-card-section>

        <q-card-actions align="right">
          <q-btn flat label="취소" color="primary" v-close-popup />
          <q-btn flat label="입력" @click="addItem" color="primary" v-close-popup />
        </q-card-actions>
      </q-card>
    </q-dialog>
  </div>
</template>

<style>
</style>

<script>
export default {
  name: "PageIndex",
  watch: {
    additionalElementType (prev, curr) {
      this.subItemQuantity = 1;
      this.subItemUnitPrice = this.additionalOptions.find((item) => item.name === prev).price;
    }
  },
  data () {
    return {
      confirm: false,
      elementType: null,
      additionalElementType: null,
      t: null,
      w: null,
      l: null,
      showWeight: false,
      showBottomProcess: false,
      negotiation: null,
      quantity: null,
      subItemQuantity: null,
      unitPrice: null,
      subItemUnitPrice: null,
      options: [
        "상고정판(고정설치판)",
        "히터판보조판(받침판)",
        "상원판",
        "하원판",
        "다리1",
        "다리2",
        "상밀판",
        "하밀판",
        "하고정판",
        "네고"
      ],
      additionalOptions: [
        { name: "E/B M12", price: 10000 },
        { name: "E/B M24", price: 20000 },
        { name: "E/B M30", price: 25000 },
        { name: "E/B M64", price: 45000 },
        { name: "포켓가공", price: null },
        { name: "NC가공", price: null },
        { name: "추가가공", price: null },
        { name: "수정가공", price: null },
        { name: "기준면(2면)", price: null },
        { name: "기준면(4면)", price: null }
      ],
      columns: [
        { name: "품명", align: "center", label: "품명", field: "name" },
        {
          name: "규격",
          align: "center",
          label: "규격",
          field: "specification"
        },
        { name: "수량", align: "center", label: "수량", field: "quantity" },
        {
          name: "가공 내역",
          align: "center",
          label: "가공 내역",
          field: "processingHistory"
        },
        { name: "면적", align: "center", label: "면적", field: "area" },
        { name: "무게", align: "center", label: "무게", field: "weight" },
        {
          name: "단가",
          align: "center",
          label: "단가",
          field: "unitPrice"
        },
        { name: "금액", align: "center", label: "금액", field: "price" }
      ],
      data: [],
      additionalData: []
    };
  },
  computed: {
    priceTotal () {
      return this.data.reduce((prev, curr) => prev + curr.price, 0);
    }
  },
  methods: {
    addItem () {
      if (this.elementType === "네고") {
        this.data.push({
          name: this.elementType,
          price: this.unitPrice * -1
        });
        return;
      }
      const fourSideArea = (this.w + this.l) * 2 * this.t * 0.01 * this.quantity;
      const twoSideArea = this.w * this.l * 2 * 0.01 * this.quantity;
      const sixSideArea = fourSideArea + twoSideArea;
      // const weight = this.t * this.w * this.l * 0.00000785 * this.quantity;
      const sojaeWeight =
        (this.t + 10) *
        (this.w + 10) *
        (this.l + 10) *
        0.00000785 *
        this.quantity;
      this.data.push({
        name: this.elementType,
        specification: `${this.t} * ${this.w} * ${this.l}`,
        quantity: this.quantity,
        processingHistory: "6면삭",
        area: sixSideArea,
        weight: "",
        unitPrice: this.unitPrice,
        price: sixSideArea * this.unitPrice
      });
      if (this.showBottomProcess) {
        this.data.push({
          name: "",
          specification: "",
          quantity: "",
          processingHistory: "연마",
          area: twoSideArea,
          weight: "",
          unitPrice: 5,
          price: twoSideArea * 5
        });
      }
      if (this.showWeight) {
        this.data.push({
          name: "소재비",
          specification: `${this.t + 10} * ${this.w + 10} * ${this.l + 10}`,
          quantity: this.quantity,
          processingHistory: "",
          area: "",
          weight: sojaeWeight,
          unitPrice: 1070,
          price: sojaeWeight * 1070
        });
      }
      if (this.additionalData.length > 0) {
        this.additionalData.forEach(item => {
          this.data.push({
            name: "",
            specification: "",
            quantity: item.quantity,
            processingHistory: item.name,
            area: "",
            weight: "",
            unitPrice: item.unitPrice,
            price: item.unitPrice * item.quantity
          });
        });
      }
      this.additionalData.length = 0;
      this.showWeight = false;
    },
    addSubItem () {
      this.additionalData.push({
        name: this.additionalElementType,
        unitPrice: this.subItemUnitPrice,
        quantity: this.subItemQuantity
      });
    }
  }
};
</script>
