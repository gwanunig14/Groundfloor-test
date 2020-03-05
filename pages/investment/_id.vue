<template>
  <b-container>
    <h1 class="title">LOAN</h1>
    <p>Address: {{ investment.address }}</p>
    <p>Purpose: {{ investment.purpose }}</p>

    <section v-if="!investment.fully_funded" class="investment_form">
      <h3 class="subtitle">Your Investment?</h3>
      <b-form @submit="onSubmit">
        <b-form-group label="Loan Amount" label-for="loan_amount_dollars">
          <p>Fully fund this loan with: ${{ remaining }}</p>
          <b-input-group>
            <b-input-group-prepend>
              <span class="input-group-text">$</span>
            </b-input-group-prepend>
            <b-form-input
              id="investment_amount_dollars"
              v-model="investment_amount_dollars"
              type="number"
              name="investment_amount_dollars"
            />
          </b-input-group>
        </b-form-group>

        <b-button type="submit" variant="primary">Submit</b-button>
      </b-form>
    </section>

    <section class="investment_funds">
      <h3 class="subtitle">Investments</h3>
      <ul>
        <ul class="funds-list">
          <li v-for="investment in funds" :key="investment.id">
            <p>Amount: ${{ investment.amount }}</p>
          </li>
        </ul>
      </ul>
      <p>Total funds: ${{ total }} of ${{ investment.loan_amount_dollars }}</p>
    </section>

    <p>
      <b-button type="button" vairant="success" @click="goBack">Back</b-button>
    </p>
  </b-container>
</template>
<script>
export default {
  async asyncData({ $axios, params }) {
    let investment = await $axios.get(`/api/investment/${params.id}`)
    let funds = await $axios.get(`/api/investment/${params.id}/funds`)
    let funding_total = 0
    if (funds.data.length) {
      funds.data.forEach(fund => {
        funding_total += fund.amount
      })
    }
    return {
      investment: investment.data,
      funds: funds.data,
      total: funding_total,
      investment_amount_dollars: 0,
      remaining: investment.data.loan_amount_dollars - funding_total
    }
  },
  methods: {
    async onSubmit(ev) {
      ev.preventDefault()
      let { investment, investment_amount_dollars, total } = this
      if (
        investment_amount_dollars &&
        parseInt(investment_amount_dollars) + total <=
          investment.loan_amount_dollars
      ) {
        let amount = investment_amount_dollars
        let investment_id = investment.id
        let b = {
          investment_id,
          amount
        }
        let invest = await this.$axios({
          method: 'post',
          url: '/api/funding',
          data: b
        })

        if (parseInt(amount) + total === investment.loan_amount_dollars) {
          let invest = await this.$axios({
            method: 'put',
            url: '/api/funding/complete',
            data: { investment_id }
          })
        }
        this.$router.back()
      }
    },
    goBack() {
      this.$router.back()
    }
  }
}
</script>
