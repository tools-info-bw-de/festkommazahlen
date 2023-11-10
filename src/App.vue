<script setup>
import "bootstrap/dist/css/bootstrap.min.css"
import "bootstrap"

import { reactive, ref, computed, watch } from "vue"

const q = ref(4)

const state = reactive({
  input: "",
  output: "",
  q: 4,
  binToDec: true,
  inputInvalid: false,
  qAndInputIncompatible: false,
  explainText: "",
})

state.output = computed(() => {
  if (state.binToDec) {
    return calcAndExplainBinToDec()
  } else {
    return calcAndExplainDecToBin()
  }
})

watch(q, (newVal, oldVal) => {
  if (newVal == "" || !Number.isInteger(newVal)) {
    q.value = 0
  } else if (newVal > 16) {
    q.value = 16
  } else if (newVal < 0) {
    q.value = 0
  }
})


function calcAndExplainBinToDec() {
  let input = state.input.replace(/\s/g, "")
  let output = 0
  state.explainText = ""

  if (input.length == 0 || input.length % 8 != 0) {
    state.inputInvalid = false
    return
  }

  if (input.length > 16 || input.match(/[^01]/g)) {
    state.inputInvalid = true
    return
  }

  if (input.length < q.value) {
    state.inputInvalid = true
    state.qAndInputIncompatible = true
    return
  }

  state.inputInvalid = false

  state.explainText += `Wegen Q${q.value} sind die letzten ${q.value} Bits die Nachkommastellen.\n\n  `

  const lengths = []
  for (let i = 0; i < input.length; i++) {
    let pow = (input.length - q.value - i - 1) >= 0 ? (input.length - q.value - i - 1) : "(" + (input.length - q.value - i - 1) + ")"
    let nextPart = input[i] + " \u2219 2^" + pow
    lengths.push(nextPart.length)
    state.explainText += nextPart
    if (i != input.length - 1) {
      state.explainText += " + "
    }
  }
  state.explainText += "\n= "

  for (let i = 0; i < input.length; i++) {
    if (input[i] == "1") {
      let result = Math.pow(2, input.length - q.value - i - 1)

      output += result

      state.explainText += " ".repeat(lengths[i] - result.toString().length)
      state.explainText += result

      if (i != input.length - 1) {
        state.explainText += " + "
      }
    } else {
      state.explainText += " ".repeat(lengths[i] + 3)
    }
  }
  state.explainText += "\n= " + output

  return output
}

function calcAndExplainDecToBin() {

}


</script>

<template>
  <h1 class="mb-4">Festkommazahlen Rechner</h1>
  <div class="row">
    <div class="ioBox col-md-12 col-lg-5 d-flex flex-column">
      <div class="labelInputOutput">Eingabe</div>
      <div class="d-flex flex-row align-items-center">
        <div v-if="state.binToDec" class="inputOutputBase">
          Binär
        </div>
        <div v-else class="inputOutputBase">
          Dezimal:
        </div>

        <div class="input-group input-group-lg ms-1 q-group">
          <span class="input-group-text">Q</span>
          <input class="form-control" type="number" step="1" min="0" max="16" v-model="q">
        </div>
        <div class="ms-2 flex-fill d-flex flex-column">


          <div class="input-group-lg">
            <!-- <div class="form-floating"> -->
            <input type="text" class="form-control" :class="{ redBorder: state.inputInvalid }" id="input"
              placeholder="Eingabe" autofocus v-model="state.input"><!-- input-lg -->
            <!-- <label for="input">Eingabe</label>
            </div> -->
          </div>

          <div class="inputInvalidMessage" v-if=state.inputInvalid>
            <div v-if="state.qAndInputIncompatible">Q muss kleiner oder gleich der Eingabelänge sein.</div>
            <div v-else>Eingabe fehlerhaft!</div>
          </div>
        </div>
      </div>
      <div v-if="state.binToDec" class="explainBinInput">
        <ul>
          <li>
            Dieser Rechner kann Q0 bis Q16 berechnen.
          </li>
          <li>
            Die Binäreingabe muss 8 oder 16 Bit lang sein.
          </li>
          <li>
            Die Zahl muss positiv sein.
          </li>
        </ul>
      </div>
    </div>
    <div class="col-md-12 col-lg-2 d-flex align-items-center justify-content-center">
      <button type="button" class="btn btn-primary my-3 d-flex align-items-center" @click.prevent="switchBinDec()">
        <div class="me-2">Vertauschen</div>
        <img src="./assets/arrow-right-arrow-left-solid.svg" width="30" height="30">
      </button>
    </div>
    <div class="ioBox col-sm-12 col-lg-5 d-flex flex-column">
      <div class="labelInputOutput">Ausgabe</div>
      <div class="d-flex flex-row">
        <div v-if="!state.binToDec" class="inputOutputBase">
          Binär:
        </div>
        <div v-else class="inputOutputBase">
          Dezimal:
        </div>
        <div class="ms-2 flex-fill d-flex flex-column">
          <textarea class="form-control output" :value="state.output" cols="30" rows="2" readonly></textarea>
        </div>
      </div>
    </div>
  </div>
  <div class="row mt-5 ms-3 mb-2 explainHeader">
    Erklärung:
  </div>
  <div class="row">
    <div class="col">
      <textarea class="form-control explainTextarea" :value="state.explainText" cols="30" rows="15" readonly></textarea>
    </div>
  </div>
</template>

<style scoped>
.q-group {
  min-width: 140px;
  max-width: 140px;
}


.explainHeader {
  font-size: 1.2rem;
}

.input-lg {
  height: 75px;
}

.explainBinInput {
  text-align: left;
  margin-top: 10px;
}

.inputOutputBase {
  font-size: 1.5rem;
  font-weight: bold;
  align-self: center;
}

.explainTextarea {
  font-family: monospace;
  white-space: pre;
  background-color: #f0f0f0;
  height: 100%;
}

.invisible {
  visibility: hidden;
}

.redBorder {
  border-color: #842029;
  border-width: 2px;
}

.inputInvalidMessage {
  align-self: flex-start;
  background-color: #f8d7da;
  color: #842029;
  padding: 5px;
  border-radius: 5px;
}

.inputInvalid {
  border-color: #842029;
}

input,
select {
  border-color: gray;
}

input {
  font-size: 1.5rem !important;
}

.output {
  font-size: 1.5rem;
}

.ioBox {
  background-color: rgb(228, 228, 228);
  border-radius: 15px;
  padding: 10px;
}

.labelInputOutput {
  font-size: 1.5rem;
  font-weight: bold;
  margin-bottom: 10px;
}
</style>
