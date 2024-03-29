<template>
  <table v-if="input">
    <tbody>
    <tr v-for="weekAgo in weekRange">
      <td
        v-for="day in weekUntil(weekAgo)"
        :style="`background-color: ${rgb(day)}`"
      >
        <v-tooltip style="width: 100%" bottom>
          <div slot="activator">&nbsp;</div>
          <span>{{ tooltipText(day) }}</span>
        </v-tooltip>
      </td>
    </tr>
    </tbody>
  </table>
</template>

<script>
import moment from 'moment'
import utils from '~/plugins/utils'
import firebase from '~/plugins/firebase'

export default {
  props: {
    input: Array,
    log: firebase.firestore.DocumentSnapshot
  },
  data() {
    return {
      weekRange: utils.range(7)
    }
  },
  computed: {
    logColor() {
      if (this.log) {
        const code = this.log.data().color
        return {
          r: parseInt(code.substring(1, 3), 16),
          g: parseInt(code.substring(3, 5), 16),
          b: parseInt(code.substring(5, 7), 16)
        }
      }
    }
  },
  methods: {
    /**
     * ポップアップ表示する文字列を返す
     * @param {moment.Moment} day
     * @returns {String}
     */
    tooltipText(day) {
      const dataInput = this._createDataFromInput()
      const dateString = day.format('YYYY-MM-DD')
      let count = 0
      if (dateString in dataInput) {
        count = dataInput[dateString]
      }
      return day.format('YYYY/MM/DD') + '...' + count.toString() + this.log.data().unit
    },
    /**
     * n週間前の7日間のmomentオブジェクトの配列を返す
     * @param n
     * @returns {moment.Moment[]}
     */
    weekUntil(n) {
      return utils.range(7).map(i => {
        return moment().add(n * -7, 'days').add(i * -1, 'days')
      })
    },
    /**
     * this.inputから{dateString: countの合計}のオブジェクトを返す
     * @private
     * @returns {Object}
     */
    _createDataFromInput() {
      const r = {}
      const commits = this.input ? this.input.map(doc => doc.data()) : []
      commits.forEach(data => {
        const dateString = moment(data.date).format('YYYY-MM-DD')
        if (dateString in r) {
          r[dateString] += data.count
        } else {
          r[dateString] = data.count
        }
      })
      return r
    },
    /**
     * tdのループの中で得られるmomentオブジェクトと
     * this._createDataFromInput()の値をもとにrgbカラーを返す
     * @param date
     * @returns {string}
     */
    rgb(date) {
      const dataInput = this._createDataFromInput()
      const maxCountInput = utils.max(Object.values(dataInput))
      const dateString = date.format('YYYY-MM-DD')
      if (dateString in dataInput) {
        const currentData = dataInput[dateString]

        const color = this.logColor
        const colorRate = (currentData / maxCountInput).toFixed(2)

        return `rgba(${color.r}, ${color.g}, ${color.b}, ${colorRate})`
      }
      return 'white'
    }
  }
}
</script>

<style scoped>
table {
  width: 100%;
  max-width: 600px;
  background-color: #bcbcbc;
}

table td {
  border: solid 1px #eeeeee;
}
</style>
