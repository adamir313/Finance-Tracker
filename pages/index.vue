<template>
    <section class="flex items-center justify-between mb-10">
        <h1 class="text-4xl font-extrabold">Summary</h1>
        <div class="">
            <USelectMenu v-model="selectedView" :options="transactionViewOptions" />
        </div>
    </section>

    <section class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 sm:gap-16 mb-10">
        <Trend color="green" title="Income" :amount="incomeTotal" :last-amount="3000" :loading="isLoding" />
        <Trend color="green" title="Expense" :amount="expenseTotal" :last-amount="5000" :loading="isLoding" />
        <Trend color="red" title="Investments" :amount="7500" :last-amount="4500" :loading="isLoding" />
        <Trend color="green" title="Savings" :amount="3000" :last-amount="8000" :loading="isLoding" />
    </section>

    <section class="flex justify-between mb-10">
        <div>
            <h2 class="text-2xl font-extrabold">Transactions</h2>
            <div class="text-gray-500 dark:text-gray-400">
                You have {{ incomeCount }} incomes and {{ expenseCount }} expense this period
            </div>

        </div>
        <div>
            <TransactionModal v-model="isOpen"/>
            <UButton icon="i-heroicons-plus-circle" color="white" variant="solid" label="Add" @click="isOpen = true" />
        </div>
    </section>

    <section v-if="!isLoding">
        <div v-for="(transactionsOnDay, date) in transactionsGroupedByDate" :key="date" class="mb-10">
            <DailyTransactionSummary :date="date" :transactions="transactionsOnDay" />
            <Transaction v-for="transaction in transactionsOnDay" :key="transaction.id" :transaction="transaction"
                @deleted="refreshTransactions()" />
        </div>
    </section>
    <section v-else>
        <USkeleton class="h-8 w-full mb-2" v-for="i in 4" :key="i" />
    </section>
</template>

<script setup>
import { transactionViewOptions } from "~/constants";
const selectedView = ref(transactionViewOptions[1]);

const supabase = useSupabaseClient();

const transactions = ref([]);
const isLoding = ref(false);
const isOpen = ref(false)

const income = computed(() =>
    transactions.value.filter((t) => t.type === "Income")
);

const expense = computed(() =>
    transactions.value.filter((t) => t.type === "Expense")
);

const incomeCount = computed(() => income.value.length);
const expenseCount = computed(() => expense.value.length);

const incomeTotal = computed(() =>
    income.value.reduce((sum, transaction) => sum + transaction.amount, 0)
);

const expenseTotal = computed(() =>
    expense.value.reduce((sum, transaction) => sum + transaction.amount, 0)
);

const fetchTransactions = async () => {
    isLoding.value = true;
    try {
        const { data } = await useAsyncData("transactions", async () => {
            const { data, error } = await supabase.from("transactions").select();

            if (error) return [];

            return data;
        });

        return data.value;
    } finally {
        isLoding.value = false;
    }
};

const refreshTransactions = async () =>
    (transactions.value = await fetchTransactions());

await refreshTransactions();

const transactionsGroupedByDate = computed(() => {
    let grouped = {};

    for (const transaction of transactions.value) {
        const date = new Date(transaction.created_at).toISOString().split("T")[0];

        if (!grouped[date]) {
            grouped[date] = [];
        }

        grouped[date].push(transaction);
    }

    return grouped;
});
</script>
