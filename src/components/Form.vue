<template>
    <div class="mortgage-form-block">
        <h3>Mortgage Calculator</h3>
        <div>
            <form ref="form" @submit.prevent="submitForm">
                <div class="row">
                    <div class="form-field col-md">
                        <span>Property Purchase Price</span><br>
                        <CurrencyInput v-model="purchasePrice" :options="{ currency: 'EUR' }"
                            placeholder="Enter Purchase Price"
                            @input="changeRange($event.target.value, this.totalSaving, this.commission, this.repaymentRate)" />
                        <br>
                        <span v-if="formSubmitted && this.purchasePrice === 0 || this.purchasePrice === null"
                            class="error-message">enter purchase
                            price</span>
                    </div>
                    <div class="form-field col-md">
                        <span>Total savings</span><br>
                        <CurrencyInput v-model="totalSaving" :options="{ currency: 'EUR' }"
                            placeholder="Enter your Total savings"
                            @input="changeRange(this.purchasePrice, $event.target.value, this.commission, this.repaymentRate)" />
                        <br>
                        <span v-if="formSubmitted && this.totalSaving === 0 || this.totalSaving === null"
                            class="error-message">enter total Saving</span>
                    </div>
                </div>
                <div class="row ">
                    <div class="form-field col-md">
                        <span>Real estate commission</span><br>
                        <div class="commission-block">
                            <div class="radio-block">
                                <label class="radio-label" :class="this.commission ? 'active-button' : ''">
                                    <span>Yes</span>
                                    <input type="radio" v-model="commission" v-bind:value=true
                                        @input="changeRange(this.purchasePrice, this.totalSaving, true, this.repaymentRate)"
                                        class="custom-radio" />
                                </label>

                            </div>
                            <div class="radio-block">
                                <label class="radio-label" :class="!this.commission ? 'active-button' : ''">
                                    <span>No</span>
                                    <input type="radio" v-model="commission" v-bind:value=false
                                        @input="changeRange(this.purchasePrice, this.totalSaving, false, this.repaymentRate)"
                                        class="custom-radio" />
                                </label>

                            </div>

                        </div>
                        <div v-if="formSubmitted && this.commission === null" class="error-message">please
                            select commission</div>
                    </div>
                    <div class="form-field col-md">
                        <span>Annual repayment rate(%)</span><br>
                        <span class="repayment-handler" v-on:click="() => decrement()">-</span>
                        <input disabled type="text" class="repayment-input" v-model="repaymentRate"
                            value={{repaymentRate}} />
                        <span class="repayment-handler" v-on:click="() => increment()">+</span>
                        <div v-if="formSubmitted && this.repaymentRate === 0" class="error-message">please enter
                            Interest</div>
                    </div>
                </div>
                <div class="submit-block">
                    <input class="submit" type="submit" value="Calculate">
                </div>
            </form>
        </div>
    </div>
    <div class="live-board-block">
        <div class="row">
            <div class="live-board col-md">
                <span>Implied-loan</span><br>
                <span class="live-values">
                    {{ rawLoanAmount.toLocaleString("de-DE", { style: "currency", currency: "EUR" }) }}</span>

            </div>
            <div class="live-board  col-md">
                <span>Loan to value</span><br>
                <span class="live-values">{{ loanToValue.toFixed(2) }} %</span>
            </div>
        </div>
    </div>
</template>
<script>
import CurrencyInput from "./CurrencyInput.vue";
import { brokerTax, cityTax } from '../utils/Constants';
export default {
    name: 'Form',
    components: {
        CurrencyInput,
    },
    data() {
        return {
            purchasePrice: 0,
            totalSaving: 0,
            commission: null,
            repaymentRate: 0,
            formSubmitted: false,
            rawLoanAmount: 0,
            loanToValue: 0,
            formValid: false,
            mortgageData: [],
            user: "Arun"
        };
    },
    methods: {
        submitForm: function () {

            this.formSubmitted = true;
            if (this.formValid) {
                const request = {
                    query: {
                        root: {
                            rates_table: {
                                property_price: this.purchasePrice,
                                repayment: this.repaymentRate,
                                loan_amount: this.rawLoanAmount,
                                years_fixed: [5, 10, 15, 20, 25, 30]
                            }
                        }
                    }
                }
                const formattedRequest = JSON.stringify(request)
                fetch("https://hypofriend.de/q", {
                    "headers": {
                        "accept": "*/*",
                        "accept-language": "en,de;q=0.9,cs;q=0.8",
                        "cache-control": "no-cache",
                        "content-type": "application/json",
                        "pragma": "no-cache",
                        "sec-fetch-dest": "empty",
                        "sec-fetch-mode": "cors",
                        "sec-fetch-site": "same-origin",
                    },
                    "referrer": "https://hypofriend.de",
                    "referrerPolicy": "same-origin",
                    "body": formattedRequest,
                    "method": "POST",
                    "mode": "cors",
                    "credentials": "include"
                }).then(response => response.json()).then(data => console.log(data)).catch((error) => console.log(error));
            }

        },
        changeRange(purchase_price, savings, commission) {
            const price = parseInt(purchase_price, 10)
            const savingAmount = parseInt(savings, 10)
            const notaryCosts = price > 0 ? (2144.0 + (0.013 * (price - 100000.0))) : 0
            const brokerCosts = commission ? brokerTax * price : 0;
            const stampDutyCosts = cityTax * price;
            const totalCost = notaryCosts + brokerCosts + stampDutyCosts;
            const loanAmount = (totalCost - savingAmount) + price;
            this.rawLoanAmount = loanAmount;
            this.loanToValue = loanAmount / price;
            if (this.purchasePrice > 0 && this.totalSaving > 0 && this.commission !== null && this.repaymentRate > 0) {
                this.formValid = true;
            }
        },
        increment() {
            this.repaymentRate++;
            this.changeRange(this.purchasePrice, this.totalSaving, this.commission, this.repaymentRate);
        },
        decrement() {
            this.repaymentRate > 0 ? this.repaymentRate-- : this.repaymentRate = 0;
            this.changeRange(this.purchasePrice, this.totalSaving, this.commission, this.repaymentRate);
        }
    },
};
</script>
<style>
.mortgage-form-block {
    background: white;
    border-radius: 3px;
    padding: 25px;
    text-align: center;
    font-size: 18px;
    font-weight: bold;
    margin-bottom: 1rem;
    box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.2), 0 6px 20px 0 rgba(0, 0, 0, 0.19);
}

.repayment-handler {
    background-color: #adb5bd;
    padding: 8px;
    margin: 0;
    height: 38px;
    display: inline-block;
    width: 40px;
    cursor: pointer;
}

.commission-block {
    background-color: #F0f0f0;
    padding: 4px;
    width: fit-content;
    border: 2px solid rgba(0, 0, 0, 0.2);
    border-radius: 4px;
    min-width: 250px;
    display: inline-block;
}

.error-message {
    color: red;
    text-align: center;
    font-size: 12px;
    font-weight: bold;
}

.repayment-input {
    width: 200px;
}

.live-values {
    font-size: 24px;

}

.radio-block {
    position: relative;
    display: inline-block;
}

.live-board-block {
    margin: 1rem;
}

.live-board {
    background: white;
    border-radius: 3px;
    padding: 12px;
    text-align: left;
    font-weight: bold;
    margin-bottom: 1rem;
    width: 200px;
    display: inline-block;
    box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.2), 0 6px 20px 0 rgba(0, 0, 0, 0.19);

    @media (min-width: 700px) {
        margin: 1rem;
    }
}

.live-board span {
    font-size: 12px;
    font-weight: 700;
    text-align: left;
}

.radio-label {
    width: 100px;
    font-size: 12px;
    background-color: transparent;
    text-align: center;
    display: inline-block;
    cursor: pointer;
    padding: 8px;
}

.active-button {
    background-color: #fff;
    color: #1b8391;
    width: 100px;
    border-radius: 4px;
    box-shadow: 0 2px 4px 0 rgba(0, 0, 0, 0.2);
}

.custom-radio {
    width: 100px;
    position: absolute;
    left: 0;
    visibility: hidden;
}

.submit-block {
    border-top: 2px solid #adb5bd;
    padding: 10px;
    margin-top: 20px;
}

.submit {
    background-color: #1b8391;
    color: #fff;
    border: 1px solid #1b8391;
    width: 180px;
}

input:checked {
    color: #1b8391;
}

.form-field {
    display: inline-block;
}

input {
    padding: 12px;
    border-radius: 4px;
    border: none;
    background-color: #F0f0f0;
    border: 2px solid rgba(0, 0, 0, 0.2);
}

button {
    border: 2px solid rgba(0, 0, 0, 0.2);
    cursor: pointer;
    font-size: 18px;
    color: #1b8391;
    background-color: #fff;
    box-shadow: 0 2px 4px 0 rgba(0, 0, 0, 0.2);
    width: 40px;
    height: 45px;
}

span {
    font-size: 14px;
    margin: 4px;
    font-weight: 500;
    width: 50%;
    padding: 4px;
}
</style>