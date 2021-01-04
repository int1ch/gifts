<template>
  <v-app>
    <v-app-bar app color="primary" dark>
      <div class="d-flex align-center">
        <v-icon>mdi-gift-outline</v-icon>
        <div class="ml-2">Найди КОД</div>
      </div>
    </v-app-bar>

    <v-main>
      <v-container>
        <v-row>
          <v-col>
            <v-btn
              :icon="selectedLine !== gift.id"
              fab
              :color="giftColor(gift)"
              v-for="gift in gifts"
              :key="gift.id"
              @click="selectLine(gift.id)"
              small
            >
              <v-icon dark>mdi-gift-outline</v-icon>
            </v-btn>
          </v-col>
        </v-row>
        <v-row>
          <v-col>
            <v-text-field
              v-model="inputCode"
              label="Введи код"
              prepend-inner-icon="mdi-magnify"
              solo
              v-on:keyup.enter="searchCode()"
            >
              <template v-slot:prepend>
                Введи код
              </template>
            </v-text-field>
          </v-col>
        </v-row>
        <v-row v-if="error">
          <v-col>
            <v-alert border="top" color="red lighten-2" dark>
              {{ error }}
            </v-alert>
          </v-col>
        </v-row>
        <v-row v-for="code in codes" :key="code.code">
          <v-col cols="4">
            {{ code.code }}
          </v-col>
          <v-col cols="8">
            {{ code.text }}
          </v-col>
        </v-row>
        <v-row v-if="!selectedLine">
          <p>
            Привет, ваша задача найти коды по подсказками, ввести их в поле
            вверху получить новую подсказку и где то там вас ждет ...
          </p>
          <p>
            И для начал выбирайте подарок
            <v-icon @click="selectLine('next')">mdi-gift-outline</v-icon>
          </p>
        </v-row>
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
import Vue from "vue";

function ruTransform(input) {
  const charsFrom = "УКЕНЗХЁВАРОСМТO";
  const charsToTo = "YKEH3XEBAP0CMT0";
  const charsFromL = "укенгзхёвапросмтo";
  const charsToToL = "YKEHR3XEBANP0CMT0";
  console.log("Transform:", input);

  let testCode = input.toUpperCase().replace(/[А-ЯЁёOo\s]/g, ch => {
    if (ch === " " || ch === "\t") {
      return "";
    }
    return (
      charsToTo.charAt(charsFrom.indexOf(ch)) ||
      charsToToL.charAt(charsFromL.indexOf(ch)) ||
      ch
    );
  });
  console.log("transoformed:", testCode, testCode.toLowerCase());
  return testCode;
}

export default {
  name: "App",

  components: {},

  data: () => ({
    inputCode: "",
    selectedLine: null,
    isLoading: false,
    error: null,
    gifts: [
      { id: "A", status: "indoor" },
      { id: "Y", status: "indoor" },
      { id: "P", status: "indoor" },
      { id: "X", status: "free" },
      { id: "Z", status: "free", lastCode: "" },
      { id: "S", status: "free" },
      //{ id: "H", status: "free" },
      { id: "E", status: "free" }
    ],
    codes: [
      //  { code: "START", text: "ну что ж поехали" },
      //  { code: "END", text: "Всё вы нашли" },
    ],
    lines: {
      X: [
        { code: "START", text: "введи номер черной машины" },
        { code: "X514PB190", text: "номер контейнера" },
        {
          code: "WSCU309671",
          text: "под южной западной опорой контейнера кулечек найдеш"
        }
      ],
      Z: [
        { code: "START", text: "введи номер серебристой машины" },
        { code: "B627ОM190", text: "иди в поле, код на столбе" },
        { code: "MТP2683", text: "копай под опорой столба" }
      ],
      Y: [
        { code: "START", text: "посмотри код за холодильником" },
        { code: "Y298", text: "на зеленой тарелке" },
        { code: "Y455", text: "на самой высокой полке" }
      ],
      P: [
        { code: "START", text: "найди код, на ножке стола" },
        { code: "P45", text: "посмотри за печкой" },
        {
          code: "P89",
          text: "в комнате где иногда мерзко визжит злобный зверь"
        }
      ],
      S: [
        { code: "START", text: "на двери контейнера внутри на S" },
        { code: "S15", text: "под лестницей" },
        { code: "S350", text: "Северный угол дома" }
      ],
      /* H: [
        { code: "START", text: "на двери контейнера внутри на C" },
        { code: "C10", text: "под лестницей" },
        { code: "C23", text: "на северном угле дома" },
      ],*/
      A: [
        { code: "START", text: "на входной двери" },
        { code: "A350", text: "на другой двери" },
        { code: "A10", text: "на втором этаже в северном углу дома" }
      ],
      E: [
        { code: "START", text: "на южном угле дома" },
        { code: "E15", text: "У самой большой елки" },
        { code: "E33", text: "Следуй карте" }
      ]
    }
    //
  }),
  computed: {},
  methods: {
    giftStatus(name) {
      return this.gifts.find(el => el.id === name);
    },
    searchCode() {
      const testCode = ruTransform(this.inputCode);
      console.log("IN ", this.inputCode, "GET ", testCode);
      //this.codes.unshift({ code: testCode, text: "xxx" });
      const searchCodes = this.lines[this.selectedLine];
      let matchedCode = null;

      if (searchCodes) {
        console.log("WTF", searchCodes);
        matchedCode = searchCodes.find(el => {
          console.log("CMP:", el.code, ruTransform(el.code), testCode);
          return ruTransform(el.code) === testCode;
        });
        //console.log("IN ", this.inputCode, searchCodes, "GET ", testCode, matchedCode);
        if (matchedCode) {
          this.codes.unshift(matchedCode);
          this.giftStatus(this.selectedLine).lastCode = testCode;
          this.error = "";
          this.inputCode = "";
        } else {
          this.error = "код не тот проверьте пожалуйста";
        }
      } else {
        this.error = "кажется подарок не выбран";
      }
      try {
        this.$axios.get("/code", {
          code: this.inputCode,
          line: this.selectedLine,
          testCode,
          matchedCode
        });
      } catch (e) {
        console.log("remote error");
      }
    },
    selectLine(name) {
      console.log("Select line:", name, this.selectedLine);
      if (this.selectedLine !== name) {
        let giftInfo = this.giftStatus(name);
        console.log("Gift", giftInfo);

        if (giftInfo && this.lines[name]) {
          let lastCode = giftInfo.lastCode || "START";
          let allCodes = this.lines[name];
          let lastCodeIndex = allCodes.findIndex(el => el.code === lastCode);
          console.log("CODE:", lastCode, lastCodeIndex);
          if (lastCodeIndex >= 0) {
            Vue.set(
              this,
              "codes",
              this.lines[name].slice(0, lastCodeIndex + 1).reverse()
            );
          }
        }
        //
      }
      this.selectedLine = name;
      this.error = null;
    },
    giftColor(gift) {
      switch (gift.status) {
        case "inprogress":
          return "orange";
        case "indoor":
          return "green";
        case "free":
          return "#3385ff";
        case "found":
          return "gray";
      }
      return "indigo";
    }
  }
};
</script>
