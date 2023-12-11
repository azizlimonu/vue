<script setup>
import { computed, ref, onMounted } from 'vue'
import { storeToRefs } from 'pinia'
import { showNotification } from '@/utils/notify'
import { useCounterStore } from '@/stores/counter'

const store = useCounterStore();
const { dataForm } = storeToRefs(store);

// reactive var
const greeting = ref('');
const budget = ref(0);
const budgetAmount = ref("");
const expenses = ref(0);
const remainingBudget = ref(budget.value);
const expenseAmount = ref("");
const expenseDescription = ref('');
const expenseTitle = ref('');
const expenseDate = ref('');
const expenseList = ref([]);

let editedItem = computed(() => findData(editIndex.value));
const selectedItem = ref(null);
let modalOpen = ref(false);
let editIndex = ref(null);

const modifiedArray = ref([]);

// function
const openModal = (indexes) => {
  modalOpen.value = ref(true);
  editIndex.value = indexes;
};

const closeModal = () => {
  modalOpen.value = false;
};

const findData = (indexes) => {
  modifiedArray.value = [...expenseList.value];
  selectedItem.value = modifiedArray.value.find(item => item.id === indexes);

  return selectedItem.value;
};

const addBudget = () => {
  let amount = parseFloat(budgetAmount.value);
  if (isNaN(amount) || amount <= 0) {
    showNotification({
      message: 'Please Enter Valid Amount',
      success: false
    });
    return;
  }

  remainingBudget.value += amount;
  budget.value += amount;

  showNotification({
    essage: "Budget Sucessfully Added",
    success: true,
  })

  budgetAmount.value = "";
};

const addExpense = () => {
  const amount = parseFloat(expenseAmount.value);
  if (isNaN(amount) || amount <= 0) {
    showNotification({
      message: 'Please enter a valid expense amount.',
      success: false,
    });
    return;
  }

  if (amount > remainingBudget.value) {
    showNotification({
      message: 'Insufficient Balance..',
      success: false,
    });
    return;
  }
  expenses.value += amount;
  remainingBudget.value -= amount;

  expenseList.value.push({
    id: getRandomInt(1000),
    amount,
    description: expenseDescription.value,
    title: expenseTitle.value,
    date: expenseDate.value,
  });

  showNotification({
    message: "Expense Sucessfully Added",
    success: true,
  });

  expenseAmount.value = "";
  expenseDescription.value = '';
}

const removeExpense = (index) => {
  const removedExpense = expenseList.value[index];
  expenses.value -= removedExpense.amount;
  remainingBudget.value += removedExpense.amount;

  expenseList.value.splice(index, 1);
  showNotification({
    message: "Expense Sucessfully Removed",
    success: true,
  });
};

const reset = () => {
  budget.value = 0;
  remainingBudget.value = 0;
  expenses.value = 0;
  expenseList.value.splice(0)
  showNotification({
    message: "All data Cleared",
    success: true,
  });
};

const getRandomInt = (max) => {
  return Math.floor(Math.random() * max);
}

const getMinDate = () => {
  const today = new Date();
  const year = today.getFullYear();
  let month = today.getMonth() + 1; // months are zero-indexed
  let day = today.getDate();

  // Add leading zero if needed
  month = month < 10 ? `0${month}` : month;
  day = day < 10 ? `0${day}` : day;

  return `${year}-${month}-${day}`;
}

const setGreeting = () => {
  const currentTime = new Date().getHours();

  const timeOfDay =
    currentTime >= 6 && currentTime < 12
      ? 'Morning'
      : currentTime >= 12 && currentTime < 18
        ? 'Afternoon'
        : currentTime >= 18 && currentTime < 24
          ? 'Evening'
          : 'Night';

  greeting.value = `Good ${timeOfDay}!`;
};


const minDate = getMinDate();

onMounted(() => {
  setGreeting();
});

</script>

<template>
  <main>
    <nav>
      <div class="nav-title">
        Expense Tracker
      </div>
    </nav>

    <div class="container">
      <div class="main-title flex">
        <div class="dashborad-title">
          <h1 class="title">Dashboard</h1>
          <p>Hello, {{ greeting }}</p>
        </div>

        <div class="reset-btn flex" id="reset" @click="reset">
          Reset
        </div>
      </div>

      <main class="dashboard">
        <div class="budgetSection flex">
          <div class="total-balance flex fl-col">
            <div class="balance-title">
              <p>Your Budget</p>
            </div>
            <div class="balance-amount text-elipses">
              <h1 id="balance-amount">Rp. {{ budget.toLocaleString() }}</h1>
            </div>
          </div>

          <div class="addBudget flex fl-col">
            <div class="budget-title">
              Budget
            </div>
            <form @submit.prevent="addBudget" class="flex fl-col">
              <input type="number" v-model="budgetAmount" class="input-field" id="budgetAmount"
                placeholder="Enter your budget.." autofocus>
              <button class="primary-btn flex" type="submit" id="addBudget">
                Add Budget
                <ion-icon name="arrow-forward-outline">
                </ion-icon>
              </button>
            </form>
          </div>
        </div>

        <div v-if="modalOpen">
          <div id="myModal" class="modal">
            <div class="addExpense modal-content flex fl-col">
              <div style="display:flex; justify-content:space-between;width:100%">
                <div class="expense-title">Edit Expense</div>
                <span class="close" @click="closeModal">&times;</span>
              </div>

              <form @submit.prevent="updateExpense" class="flex fl-col expenseForm">
                <div>
                  <label>Expense Title: </label>
                  <input type="text" class="input-field" id="expTitle" v-model="editedItem.title"
                    placeholder="Enter your expense title.." required maxlength="15" autocomplete="off">
                </div>

                <div>
                  <label>Expense Description: </label>
                  <input type="text" class="input-field" id="expDesc" v-model="editedItem.description"
                    placeholder="Enter your expense Desc.." required maxlength="30" autocomplete="off">
                </div>

                <div>
                  <label>Expense Amount: </label>
                  <input type="number" class="input-field" v-model="editedItem.amount" id="expAmount"
                    placeholder="Enter your expense.." required autocomplete="off">
                </div>

                <div>
                  <label>Current Date: </label>
                  <input type="date" class="input-field" v-model="editedItem.date" id="date" :min="minDate"
                    placeholder="Enter your expense.." required autocomplete="off">
                </div>

                <button class="primary-btn flex" id="addExpense" type="submit">Update Expenses<ion-icon
                    name="arrow-forward-outline"></ion-icon>
                </button>
              </form>
            </div>
          </div>
        </div>

        <div class="expenseSection flex">
          <div class="expense-list-box flex fl-col">
            <div class="income-expense-boxes flex">
              <div class="incomebox flex fl-col text-elipses">
                <h2 class="income-title">Current Balance</h2>
                <h1 id="currentAmount" class="current-amount">Rp. {{ remainingBudget.toLocaleString() }}</h1>
              </div>

              <div class="expensebox flex fl-col text-elipses">
                <h2 class="expense-title">Expense</h2>
                <h1 id="expenseAmount" class="expense-amount">Rp. {{ expenses.toLocaleString() }}</h1>
              </div>
            </div>

            <div class="main-title">
              <h2>Expense List : </h2>
            </div>

            <div class="expenseList">
              <ul class="listITems flex fl-col" id="expenseList">
                <li v-for="(expense, index) in expenseList" :key="expense.id" class="item flex">
                  <div class="list-expense-title-desc text-elipses flex">
                    <div class="flex fl-col list-text">
                      <div class="list-expense-title">{{ expense.title }}</div>
                      <div class="list-expense-desc">{{ expense.description }}</div>
                    </div>
                    <div class="list-expense-amount flex fl-col">
                      <span>Rp. {{ expense.amount.toLocaleString() }}</span>
                      <div class="date">{{ expense.date }}</div>
                    </div>
                  </div>
                  <button id="edit" @click="openModal(expense.id)">Edit</button>
                  <button id="delete" @click="removeExpense(expense.id)">X</button>
                </li>
              </ul>
            </div>

          </div>

          <div class="addExpense flex fl-col">
            <div class="expense-title">Add Expense</div>
            <form @submit.prevent="addExpense" class="flex fl-col expenseForm">
              <div>
                <label>Expense Title: </label>
                <input type="text" class="input-field" id="expTitle" v-model="expenseTitle"
                  placeholder="Enter your expense title.." required maxlength="15" autocomplete="off">
              </div>
              <div>
                <label>Expense Description: </label>
                <input type="text" class="input-field" id="expDesc" v-model="expenseDescription"
                  placeholder="Enter your expense Desc.." required maxlength="30" autocomplete="off">
              </div>
              <div>
                <label>Expense Amount: </label>
                <input type="number" class="input-field" v-model="expenseAmount" id="expAmount"
                  placeholder="Enter your expense.." required autocomplete="off">
              </div>

              <div>
                <label>Current Date: </label>
                <input type="date" class="input-field" v-model="expenseDate" id="date" :min="minDate"
                  placeholder="Enter your expense.." required autocomplete="off">
              </div>

              <button class="primary-btn flex" id="addExpense" type="submit">Add Expense <ion-icon
                  name="arrow-forward-outline"></ion-icon></button>
            </form>
          </div>

        </div>
      </main>
    </div>
  </main>

  <footer class="flex container">
    <div class="footer-text">
      Copyright © 2023 - All right reserved
    </div>
  </footer>
</template>

<style scoped></style>