<template>
    <v-expansion-panels popout focusable>
        <v-expansion-panel v-for="loanOption in loanOptions" :key="loanOption.id">
            <v-expansion-panel-header>
                {{ loanOption.duration }} Months Plan - {{ loanOption.interest_rate }}% interest
                rate</v-expansion-panel-header
            >
            <v-expansion-panel-content>
                <v-text-field
                    label="Minimum Amount"
                    type="number"
                    disabled
                    v-bind:value="loanOption.minimum_amount"
                    prefix="$"
                ></v-text-field>
                <v-text-field
                    label="Maximum Amount"
                    type="number"
                    disabled
                    v-bind:value="loanOption.maximum_amount"
                    prefix="$"
                ></v-text-field>
                <v-text-field
                    label="Amount Needed"
                    type="number"
                    v-model="requested_amount"
                    v-bind:value="loanOption.minimum_amount"
                    prefix="$"
                ></v-text-field>
                <v-btn
                    class="ma-2"
                    :loading="loading"
                    :disabled="loading"
                    color="secondary"
                    @click="createLoan(loanOption)"
                >
                    Request Loan
                </v-btn>
            </v-expansion-panel-content>
        </v-expansion-panel>
    </v-expansion-panels>
</template>

<script>
import axios from "axios";

export default {
    name: "CreateLoan",
    data() {
        return {
            loanOptions: [],
            loading: false,
            requested_amount: 0.0,
        };
    },
    async mounted() {
        this.loadLoanOptions();
    },
    methods: {
        async loadLoanOptions() {
            const token = localStorage.getItem("token");

            const loanOptions = {
                method: "get",
                url: `loan-options/`,
                headers: {
                    Authorization: `Token ${token}`,
                },
            };
            try {
                const response = await axios(loanOptions);

                // Grab the data.
                let options = response.data;

                // Sort the list descending.
                options = options.sort(function(a, b) {
                    return b["duration"] - a["duration"];
                });

                // Keep the loan options to use when changing plans.
                this.loanOptions = options;
            } catch (error) {
                console.log("Error getting loan options " + error);
            }
        },
        async createLoan(loanOption) {
            console.log(loanOption.minimum_amount - 1);
            console.log(this.requested_amount < loanOption.minimum_amount - 1);
            console.log(loanOption.maximum_amount);
            console.log(this.requested_amount > loanOption.maximum_amount);
            if (
                Number(this.requested_amount) < Number(loanOption.minimum_amount - 1) ||
                Number(this.requested_amount) > Number(loanOption.maximum_amount)
            ) {
                console.log("Wrong");
                this.$toast.error("Loan option selected doesn't support amount requested.");
                return;
            }

            const token = localStorage.getItem("token");
            const user = JSON.parse(localStorage.getItem("user"));

            this.loading = true;
            const createLoan = {
                method: "post",
                url: `loans/`,
                headers: {
                    Authorization: `Token ${token}`,
                },
                data: {
                    customer: user.id,
                    amount: Number(this.requested_amount),
                    option: loanOption.id,
                },
            };

            try {
                const response = await axios(createLoan);
                console.log(response);
                this.$toast.success("Successfully Requested Loan.");
            } catch (error) {
                this.$toast.error("Creating loan Error.");
            }
            this.loading = false;
        },
    },
};
</script>
