<template>
    <div>
        <v-data-table
            :headers="fundHeaders"
            :items="funds"
            :single-expand="singleExpand"
            :expanded.sync="expanded"
            item-key="id"
            show-expand
            class="elevation-1"
        >
            <template v-slot:top>
                <v-toolbar flat>
                    <v-toolbar-title>Pending Funds</v-toolbar-title>
                    <v-spacer></v-spacer>

                    <v-toolbar-title>Bank Balance : {{ balance }}$</v-toolbar-title>
                </v-toolbar>
            </template>
            <template v-slot:expanded-item="{ headers, item }" class="text-center">
                <v-card :colspan="headers.length" style="margin:20px; padding: 20px; ">
                    <v-card-text>
                        <p>Amount Requested => {{ item.amount }}</p>
                        <p>Interest Rate => {{ item.option.interest_rate }}%</p>
                        <p>Duration => {{ item.option.duration }} Months</p>
                        <p>started => {{ item.started }}</p>
                    </v-card-text>
                    <v-card-actions>
                        <v-btn text color="blue accent-4" @click="makeDecision(item, 2)">
                            Approve
                        </v-btn>
                        <v-btn text color="red accent-4" @click="makeDecision(item, 3)">
                            Deny
                        </v-btn>
                    </v-card-actions>
                </v-card>
            </template>
        </v-data-table>
    </div>
</template>

<script>
import axios from "axios";

export default {
    name: "Funds",
    data() {
        return {
            funds: [],
            fundOptions: [],
            balance: 0.0,
            fundHeaders: [
                {
                    text: "Amount",
                    align: "start",
                    sortable: true,
                    value: "amount",
                },
                { text: "Started", value: "started" },
                { text: "Status", value: "status" },
                { text: "", value: "data-table-expand" },
            ],
            fundStatus: {
                1: "PENDING",
                2: "APPROVED",
                3: "DENIED",
            },
            expanded: [],
            singleExpand: true,
        };
    },
    async mounted() {
        this.refreshBankBalance();
        await this.loadFundOptions();
    },
    methods: {
        async loadFundOptions() {
            const token = localStorage.getItem("token");

            const fundOptions = {
                method: "get",
                url: `fund-options/`,
                headers: {
                    Authorization: `Token ${token}`,
                },
            };
            try {
                const response = await axios(fundOptions);

                // Grab the data.
                let options = response.data;

                // Sort the list descending.
                options = options.sort(function(a, b) {
                    return b["duration"] - a["duration"];
                });

                // Keep the fund options to use when changing plans.
                this.fundOptions = options;

                // Start loading the user funds
                this.loadPendingFunds();
            } catch (error) {
                console.log("Error getting fund options " + error);
            }
        },
        async loadPendingFunds() {
            console.log(this.funds);
            const token = localStorage.getItem("token");

            const loadUserFunds = {
                method: "get",
                url: `funds-pending/`,
                headers: {
                    Authorization: `Token ${token}`,
                },
            };
            try {
                let { data } = await axios(loadUserFunds);
                console.log(data);
                let formattedFunds = [];
                data.forEach((fund) => {
                    formattedFunds.push({
                        id: fund.id,
                        amount: fund.amount,
                        started: fund.started.split("T")[0],
                        status: this.fundStatus[fund.status],
                        payment_url: fund.payment_url,
                        option: this.getPlanWithID(fund.option),
                    });
                });
                this.funds = formattedFunds;
            } catch (error) {
                console.log("Error");
            }
        },
        getPlanWithID(id) {
            for (let i = 0; i < this.fundOptions.length; i++) {
                let fundOption = this.fundOptions[i];
                if (fundOption["id"] == id) {
                    return fundOption;
                }
            }
        },
        makeDecision(fund, decision) {
            console.log("Deciding on fund " + decision);
            console.log(fund);
            const token = localStorage.getItem("token");

            const applyDecisionRequest = {
                method: "patch",
                url: `funds/${fund.id}/`,
                headers: {
                    Authorization: `Token ${token}`,
                },
                data: {
                    status: decision,
                },
            };

            axios(applyDecisionRequest)
                .then((res) => {
                    console.log(res);
                    if (res.data.message) {
                        alert(res.data.message);
                    }
                    this.refreshBankBalance();
                    this.loadPendingFunds();
                })
                .catch((e) => {
                    console.log(e);
                    console.log("Error making decision!");
                });
        },

        refreshBankBalance() {
            let token = localStorage.getItem("token");

            const options = {
                method: "get",
                url: `bank-balance/`,
                headers: {
                    Authorization: `Token ${token}`,
                },
            };

            axios(options)
                .then((res) => {
                    console.log("BALANCE");
                    const totalAmount = res.data.total_amount;
                    console.log(totalAmount);
                    this.balance = totalAmount;
                })
                .catch((e) => {
                    console.log(e);
                    console.log("bank balance Error!");
                });
        },
    },
};
</script>
