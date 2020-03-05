<template>
  <b-container>
    <h2 class="title">FUNDING</h2>

    <ul class="funding-list">
      <li v-for="investment in funding" :key="investment.id" style="display: flex">
        <p style="flex: 40%">
          <nuxt-link :to="'/investment/' + investment.id">{{ investment.address }}</nuxt-link>
        </p>
        <p
          style="flex: 35%"
        >Funded: ${{ investment.amount }} of ${{ investment.loan_amount_dollars }}</p>
        <p style="flex: 10%">Rate: {{ investment.rate }}%</p>
        <p style="flex: 15%">Term: {{ investment.expected_term_months }} months</p>
      </li>
    </ul>
  </b-container>
</template>

<script>
export default {
  async asyncData({ $axios }) {
    let funding = await $axios.get('/api/funding')
    funding.data.forEach((investment, index, arr) => {
      let investment_amount = 0
      if (investment.amounts) {
        investment.amounts.split(',').forEach(fund => {
          investment_amount += parseInt(fund)
        })
      }
      arr[index].amount = investment_amount
    })
    return {
      funding: funding.data
    }
  }
}
</script>
