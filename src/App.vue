<template>
<div class="horizontal_flex">
  <div class="vertical_flex">
    <textarea v-model="sourceText" rows="10" cols="30"></textarea>

    <div class="text_block">
      <p>Введённый текст</p>
      {{ sourceText }}
    </div>
    
    <div class="text_block">
      <p>Прочитанный объект</p>
      {{ this.sourceObject }}
    </div>
  </div>

  <div class="vertical_flex">
    <textarea v-model="transformationText" rows="10" cols="30"></textarea>

    <div class="text_block">
      <p>Введённые правила</p>
      {{ transformationText }}
    </div>
    
    <div class="text_block">
      <p>Прочитанные правила</p>
      {{ transformRules }}
    </div>
  </div>
  <div class="vertical_flex">
    <div class="text_block">
      <p>Результат</p>
      {{ resultObject }}
    </div>
    <button @click="preTransform">Преобразовать</button>
  </div>
  {{ lastError }}
</div>
</template>

<script>
export default {
  data() {
    return {
      sourceText: `{\n  "prev": "true"\n}`,
      sourceObject: {},
      transformationText: `[\n ["new", "added", "new text"],\n ["to", "prev", "new"],\n ["type", "prev", "integer"],\n ["edit", "prev", "25"]]`,
      transformRules: [],
      resultObject: {},
      lastError: "No errors"
    }
  },
  methods: {
    getSource() {
      try {
        this.sourceObject = JSON.parse(this.sourceText)
        this.lastError = "No errors"
      }
      catch(e) {
        this.lastError = e
      }
    },

    getRules() {
      try {
        this.transformRules = JSON.parse(this.transformationText)
        this.lastError = "No errors"
      }
      catch(e) {
        this.lastError = e
      }
    },

    preTransform() {
      try {
        this.resultObject = this.transform(this.sourceObject, this.transformRules)
      } catch (e) {
        this.lastError = e
        console.log(e)
      }
    },

    transform(sourceObject, transformRules) {
      //Сначала меняем значения полей,
      // затем меняем типы,
      // потом перемещаем все старые поля в объект результата,
      // затем добавляем новые поля

      //Промежуточный объект
      let interimObject = JSON.parse(JSON.stringify(sourceObject))

      let resultObject = {}

      //Меняем значения полей
      for (let index = 0; index < transformRules.length; index++) {

        if (transformRules[index][0] == "edit") {
          //Объект с выбранным названием равен изменённому значению
          interimObject[transformRules[index][1]] = 
            transformRules[index][2]
        }
      }

      //Меняем типы
      for (let index = 0; index < transformRules.length; index++) {

        if (transformRules[index][0] == "type") {
          //Объект с выбранным названием равен преобразованному по типу своему значению
          interimObject[transformRules[index][1]] = 
            this.tryParse(interimObject[transformRules[index][1]],
              transformRules[index][2])
        }
      }

      //Переносим поля в новый объект
      for (let index = 0; index < transformRules.length; index++) {

        if (transformRules[index][0] == "to") {
          //Объект с новым названием равен значению объекта со старым названием
          resultObject[transformRules[index][2]] =
            interimObject[transformRules[index][1]]
        }
      }

      //Добавляем новые поля в новый объект
      for (let index = 0; index < transformRules.length; index++) {

        if (transformRules[index][0] == "new") {
          //Объект с новым названием равен новому значению поля
          resultObject[transformRules[index][1]] =
          transformRules[index][2]
        }
      }

      return resultObject
    },

    tryParse(fieldText, newType) {
      try {
        if (newType == "boolean" && fieldText == "true") {
          return true
        }
        if (newType == "boolean" && fieldText == "false") {
          return false
        }
        if (newType == "integer") {
          return parseInt(fieldText)
        }

        if (newType == "float") {
          return parseFloat(fieldText)
        }

        if (newType == "string") {
          return `${fieldText}`
        }

      } catch(e) {
        this.lastError = e
        console.log(e)
      }

      return fieldText
    }
  },
  watch: {
    sourceText() {
      this.getSource()
    },
    transformationText() {
      this.getRules()
    }
  },
  beforeMount() {
    this.getSource()
    this.getRules()
  }
}
</script>

<style scoped>
p {
  font-size: 18px;
  font-weight: 600;
}
.text_block {
  width: 100%;
  height: 226px;
  color: #000;
  font-size: 16px;
  background-color: #fff;
  border: 1px solid #000;
}
.vertical_flex {
  display: flex;
  flex-direction: column;
  width: fit-content;
  width: 450px;
}
.horizontal_flex {
  display: flex;
  flex-direction: row;
  width: fit-content;
}
textarea {
  font-size: 16px;
}
</style>
