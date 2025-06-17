<template>
  <q-dialog
    ref="dialog"
    @hide="onDialogHide"
    persistent
    autofocus
    transition-show="slide-up"
    transition-hide="slide-down"
    style="width: 600px"
  >
    <q-card class="q-dialog-plugin" style="width: 600px">
      <q-bar class="text-white bg-primary" dense>
        <div>Болжау</div>
      </q-bar>

      <q-card-section>

        <div class="q-ma-md text-center text-purple text-bold">
          y = a<sub>0</sub> + a<sub>1</sub> * x<sub>1</sub> + a<sub>2</sub> * x<sub>2</sub> + a<sub>3</sub> * x<sub>3</sub> + a<sub>4</sub> * x<sub>4</sub>,
        </div>
        <div class="q-ma-md text-center">
          <span>мұндағы a<sub>0</sub> = </span>  <span class ="text-purple text-bold">{{a0}}</span>,
          <span>a<sub>1</sub> = </span> <span class ="text-purple text-bold">{{a1}}</span>,
          <span>a<sub>2</sub> = </span> <span class ="text-purple text-bold">{{a2}}</span>,
          <span>a<sub>3</sub> = </span> <span class ="text-purple text-bold">{{a3}}</span>,
          <span>a<sub>4</sub> = </span> <span class ="text-purple text-bold">{{a4}}</span>
        </div>

        <!-- rate1 -->
        <q-input
          autofocus
          v-model="rate1"
          :model-value="rate1"
          label="Өндіріс шарттары, 1 га/тенге" dense
        />

        <!-- rate2 -->
        <q-input
          v-model="rate2"
          :model-value="rate2"
          label="Еңбек шығыны, 1 га адам/сағ" dense
        />

        <!-- rate3 -->
        <q-input
          v-model="rate3"
          :model-value="rate3"
          label="Тыңайтқыш, 1 га/кг" dense
        />

        <!-- rate4 -->
        <q-input
          v-model="rate4"
          :model-value="rate4"
          label="Комбайндар,  1000 га/дана" dense
        />
        <!---->
        <q-space></q-space>

        <q-btn class="q-ma-md full-width" label="Көрсету" color="purple" :disable="validSave()" @click="fnCalc()"></q-btn>

        <div class="text-center"> Болжама мән: <span class ="text-purple text-bold">{{res}}</span> </div>

      </q-card-section>

      <q-card-actions align="right">
        <q-btn
          no-caps
          color="primary"
          icon="cancel"
          label="Жабу"
          @click="onOKClick"
          dense
        />

      </q-card-actions>
    </q-card>
  </q-dialog>
</template>

<script>


export default {
  props: ["a0", "a1", "a2", "a3", "a4"],

  data() {
    return {
      rate1: null,
      rate2: null,
      rate3: null,
      rate4: null,
      res: null
    };
  },

  emits: [
    // REQUIRED
    "ok",
    "hide",
  ],

  methods: {

    fnCalc() {
      this.res = Number(this.a0) + Number(this.a1*this.rate1) + Number(this.a2*this.rate2) +Number(this.a3*this.rate3) +Number(this.a4*this.rate4)
      this.res = this.res.toFixed(
        2)
    },

    validSave() {
      return (!this.rate1 || !this.rate2 || !this.rate3 || !this.rate4)
    },

    // following method is REQUIRED
    // (don't change its name --> "show")
    show() {
      this.$refs.dialog.show();
    },

    // following method is REQUIRED
    // (don't change its name --> "hide")
    hide() {
      this.$refs.dialog.hide();
    },

    onDialogHide() {
      // required to be emitted
      // when QDialog emits "hide" event
      this.$emit("hide");
    },

    onOKClick() {
      this.hide();
    },

  },

  created() {
  },

};
</script>
