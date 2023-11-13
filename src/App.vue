<script setup>
import "bootstrap/dist/css/bootstrap.min.css"
import "bootstrap"

import { reactive, ref, computed, watch } from "vue"

const q = ref(4)

const state = reactive({
  input: "",
  output: "",
  //q: 4,
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

function switchBinDec() {
  let temp = (typeof state.output === "undefined") ? "" : state.output.toString()
  let qFromDec = 0
  if (temp.startsWith("Q")) {
    qFromDec = parseInt(temp.split(":")[0].slice(1))
    temp = temp.split(": ")[1]
  }
  state.binToDec = !state.binToDec
  q.value = qFromDec
  state.input = temp
}

watch(q, (newVal, oldVal) => {
  if (newVal == "") {
  } else if (!Number.isInteger(newVal)) {
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
  state.qAndInputIncompatible = false

  if (input.length == 0 || q.value === "") {
    state.inputInvalid = false
    return ""
  }

  if (input.length > 16 || input.match(/[^01]/g)) {
    state.inputInvalid = true
    return
  }

  if (input.length < q.value && input.length != 0) {
    state.inputInvalid = false
    state.qAndInputIncompatible = true
    return
  }

  state.inputInvalid = false

  state.explainText += `Wegen Q${q.value} sind die letzten ${q.value} Bits die Nachkommastellen.\n\n  `

  const lengths = []
  for (let i = 0; i < input.length; i++) {
    let pow = (input.length - q.value - i - 1) >= 0 ? (input.length - q.value - i - 1) : "(" + (input.length - q.value - i - 1) + ")"
    let nextPart = input[i] + " \u2219 2^" + pow
    lengths.push(Math.max(nextPart.length, (parseInt(input[i]) * Math.pow(2, (input.length - q.value - i - 1))).toString().length))
    state.explainText += " ".repeat(Math.max(lengths[i] - nextPart.length, 0))
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

      state.explainText += " ".repeat(Math.max(lengths[i] - result.toString().length, 0))
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
  if (state.input == "") {
    state.inputInvalid = false
    state.explainText = ""
    return ""
  }

  // verhindere, dass kommaeingabe einen Fehler zeigt:
  const commaEnding = /^[0-9]+[.,]$/
  if (commaEnding.exec(state.input) != null) {
    state.inputInvalid = false
    return ""
  }

  const numberFormat = /^[0-9]+([.,][0-9]+)?$/;
  if (numberFormat.exec(state.input) == null) {
    state.inputInvalid = true
    state.explainText = ""
    return ""
  }

  state.inputInvalid = false

  let explainText = ""
  let result = ""

  let nachkomma = parseInt(state.input.split(",")[1] || state.input.split(".")[1])
  let containsComma = (state.input.includes(",") || state.input.includes(".") && nachkomma != 0)
  if (containsComma) {
    explainText += "Die Zahl enthält Nachkommastellen.\nBetrachte die Vor- und Nachkommastellen getrennt als "
    explainText += state.input.split(",")[0].split(".")[0] + " und 0,"
    explainText += (state.input.includes(",") ? state.input.split(",")[1] : state.input.split(".")[1]) + ".\n\n"
    explainText += "1. Wandel zunächst die Stellen vor dem Komma ins Binärsystem um.\n\n"
  } else {
    explainText += "1. Die Zahl enthält keine Nachkommastellen. Wandle die Zahl wie gewohnt ins Binärsystem um.\n\n"
  }

  let vorkomma = state.input.split(",")[0].split(".")[0]
  let maxLength = vorkomma.length

  while (vorkomma > 0) {
    let rest = vorkomma % 2
    result = rest.toString() + result
    explainText += "   " + " ".repeat(maxLength - vorkomma.toString().length)
    explainText += `${vorkomma} : 2 = ${Math.floor(vorkomma / 2)}, Rest ${rest}\n`
    vorkomma = Math.floor(vorkomma / 2)
  }

  if (result == "") {
    result = "0"
  }
  if (containsComma) {
    explainText += "\n   Zwischenergebnis: " + result + "\n\n"
  }

  let qCounter = 0
  if (containsComma) {
    nachkomma = parseFloat("0." + (state.input.split(",")[1] || state.input.split(".")[1]))
    explainText += `2. Wandle die Nachkommastelle ${nachkomma} ins Binärsystem um. Gehe dazu nach folgendem Schema vor:\n`
    explainText += "   1. Multipliziere die Nachkommastelle mit 2.\n"
    explainText += "   2. Die Stelle vor dem Komma ist die nächste Nachkommastelle.\n"
    explainText += "   3. Falls bei Schritt 2 die Stelle vor dem Komma eine 1 war, dann mache daraus eine 0.\n"
    explainText += "   4. Wiederhole die Schritte 1 bis 3, bis das Ergebnis 0,0 ist oder die gewünschte Genauigkeit erreicht ist.\n\n"


    nachkomma = parseFloat("0." + (state.input.split(",")[1] || state.input.split(".")[1]))
    let nachkommastellen = []

    let maxLength = nachkomma.toString().length

    while (nachkomma != 0) {
      explainText += "      " + " ".repeat(maxLength - nachkomma.toString().length)
      explainText += nachkomma + " \u2219 2 = "
      nachkomma *= 2
      nachkommastellen.push(nachkomma)
      explainText += (nachkomma == 1 ? nachkomma.toFixed(1) : nachkomma) + "  --> Ziffer: "
      if (nachkomma >= 1) {
        result += "1"
        explainText += "1"
        nachkomma = parseFloat("0." + nachkomma.toString().split(".")[1])
      } else {
        result += "0"
        explainText += "0"
      }
      qCounter++

      let duplicatePosition = checkDuplicate(nachkommastellen)
      if (duplicatePosition != null) {
        explainText += ` !!!\n   Die letzte Nachkommastelle ${nachkommastellen[duplicatePosition.second]} kam bereits an der ${duplicatePosition.first + 1}. Stelle vor.\n`
        explainText += `   Die Binärfolge [${result.slice(result.length - nachkommastellen.length + duplicatePosition.first, result.length - 1).split("").join(", ")}] wird sich daher unendlich oft wiederholen!\n`
        qCounter += 16 - result.length
        result = fillWithDuplicates(result, duplicatePosition.second - duplicatePosition.first)
        break
      }

      explainText += "\n"

      if (nachkomma == 0) {
        explainText += "      " + " ".repeat(maxLength - 3)
        explainText += "0.0 --> Ende\n"
      } else if (result.length == 16) {
        explainText += "\n   Die maximale Genauigkeit von 16 Stellen wurde erreicht.\n"
        break
      }
    }
  }

  if (containsComma) {
    explainText += `\n3. Verbinde Zwischenergebnis und Nachkommastellen zu: ${result.slice(0, result.length - qCounter)},${result.slice(result.length - qCounter)}\n`
    explainText += `   Ersetze das Komma durch den Präfix Q${qCounter} und füge ggf. Leerzeichen ein.\n`
  }

  // insert whitespace each 8th character from the right:
  if (result.length > 8) {
    result = result.slice(0, result.length - 8) + " " + result.slice(result.length - 8)
  }

  result = "Q" + qCounter + ": " + ((result == "") ? "0" : result)

  explainText += `\n   Ergebnis: ${result}`

  state.explainText = explainText

  return result
}

function fillWithDuplicates(result, repeat) {
  let duplicate = result.slice(result.length - repeat, result.length)
  let newResult = result.slice(0, result.length - repeat)
  let i = 0
  while (newResult.length < 16) {
    newResult += duplicate[i]
    i++
    if (i == duplicate.length) {
      i = 0
    }
  }
  return newResult
}

function checkDuplicate(array) {
  let duplicate = null
  for (let i = 0; i < array.length; i++) {
    let current = array[i]
    for (let j = i + 1; j < array.length; j++) {
      if (current == array[j]) {
        duplicate = { "first": i, "second": j }
        break
      }
    }
    if (duplicate != null) {
      break
    }
  }
  return duplicate
}

</script>

<template>
  <h1 class="mb-4">Festkommazahlen Rechner</h1>
  <div class="row">
    <div class="ioBox col-md-12 col-lg-5 d-flex flex-column">
      <div class="labelInputOutput">Eingabe</div>
      <div class="d-flex flex-row align-items-start">
        <div v-if="state.binToDec" class="inputOutputBase">
          Binär
        </div>
        <div v-else class="inputOutputBase">
          Dezimal:
        </div>

        <div v-if="state.binToDec" class="input-group input-group-lg ms-1 q-group">
          <span class="input-group-text">Q</span>
          <input class="form-control" type="number" step="1" min="0" max="16" v-model="q"
            :class="{ redBorder: state.qAndInputIncompatible }">
        </div>
        <div class="ms-2 flex-fill d-flex flex-column">


          <div class="input-group-lg">
            <input type="text" class="form-control" :class="{ redBorder: state.inputInvalid }" id="input"
              placeholder="Eingabe" autofocus v-model="state.input">
          </div>

          <div class="inputInvalidMessage" v-if=state.inputInvalid>
            <div>Eingabe fehlerhaft!</div>
          </div>
        </div>
      </div>
      <div v-if="state.binToDec" class="explainInput">
        <ul>
          <li>
            Die Zahl muss positiv sein.
          </li>
          <li>
            Dieser Rechner kann Q0 bis Q16 berechnen.
          </li>
        </ul>
      </div>
      <div v-else class="explainInput">
        <ul>
          <li>
            Die Zahl muss positiv sein.
          </li>
          <li>
            Das Ergebnis wird eine maximale Genauigkeit von 16 Stellen haben.
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

.explainInput {
  text-align: left;
  margin-top: 10px;
}

.inputOutputBase {
  font-size: 1.5rem;
  font-weight: bold;
  height: 56px;
  display: flex;
  align-items: center;
}

.explainTextarea {
  font-family: monospace;
  white-space: pre;
  background-color: #f0f0f0;
  height: 100%;
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
