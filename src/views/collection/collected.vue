<template>
    <v-container fluid class="pa-6 dashboard-container">
      <v-card class="data-table-card mt-6">
        <v-card-title class="white--text">
          <v-icon class="mr-2">mdi-history</v-icon>
          Collected and Posted Payments
        </v-card-title>
        <v-card-subtitle class="grey--text text--lighten-2">
           
        </v-card-subtitle>
        
        <v-divider class="divider-line"></v-divider>
        
        <v-list dark class="pa-0 data-list">
          <div v-if="filteredPayments.length === 0" class="pa-4 text-center grey--text">
            No collected or posted payments yet.
          </div>
          <v-list-item v-for="(customer, index) in filteredPayments" :key="index" class="customer-list-item">
            <v-list-item-content>
              <div class="d-flex flex-column flex-md-row justify-space-between align-start align-md-center">
                <div class="d-flex flex-column mb-2 mb-md-0">
                  <div class="white--text font-weight-bold text-subtitle-1">{{ customer.cardname }}</div>
                  <div class="grey--text text-caption">{{ customer.cardcode }} - {{ customer.branch }}</div>
                </div>
                <div class="d-flex flex-row align-center">
                  <div class="text-right mr-4">
                    <div class="primary--text text-subtitle-2 font-weight-bold">{{ formatCurrency(customer.overdueAmount) }}</div>
                    <div class="grey--text text-caption">Amount Paid</div>
                  </div>
                  <v-chip :color="getStatusColor(customer.status)" dark>{{ customer.status }}</v-chip>
                </div>
              </div>
            </v-list-item-content>
          </v-list-item>
        </v-list>
      </v-card>
    </v-container>
  </template>
  
  <script>
  export default {
    data() {
      return {
        customers: [
          { cardcode: 'C001', cardname: 'Juan Dela Cruz', branch: 'Main', overdueAmount: 5000.00, status: 'Scheduled', collectionDate: null },
          { cardcode: 'C002', cardname: 'Maria Santos', branch: 'North', overdueAmount: 0.00, status: 'Collected', collectionDate: '2025-09-06' },
          { cardcode: 'C003', cardname: 'Pedro Reyes', branch: 'South', overdueAmount: 12500.50, status: 'Posted', collectionDate: '2025-09-05' },
          { cardcode: 'C004', cardname: 'Anna Lim', branch: 'Main', overdueAmount: 300.00, status: 'Pending', collectionDate: null },
          { cardcode: 'C005', cardname: 'Crispin Basco', branch: 'North', overdueAmount: 0.00, status: 'Collected', collectionDate: '2025-09-04' },
          { cardcode: 'C006', cardname: 'Elisa Cruz', branch: 'South', overdueAmount: 750.75, status: 'Posted', collectionDate: '2025-09-05' },
          { cardcode: 'C007', cardname: 'Robert Go', branch: 'Main', overdueAmount: 2000.00, status: 'Pending', collectionDate: null },
        ],
      };
    },
    computed: {
      filteredPayments() {
        // Sort payments by collectionDate from newest to oldest
        return this.customers.filter(customer => 
          customer.status === 'Collected' || customer.status === 'Posted'
        ).sort((a, b) => {
          if (a.collectionDate && b.collectionDate) {
            return new Date(b.collectionDate) - new Date(a.collectionDate);
          }
          return 0;
        });
      },
    },
    methods: {
      getStatusColor(status) {
        if (status === 'Scheduled') return '#42a5f5';
        if (status === 'Overdue') return '#e53935';
        if (status === 'Collected') return '#43a047';
        if (status === 'Pending') return '#ffb300';
        if (status === 'Posted') return '#673ab7';
        return '#616161';
      },
      formatCurrency(amount) {
        if (amount === null || amount === undefined) return '₱ 0.00';
        return `₱ ${amount.toFixed(2).replace(/\B(?=(\d{3})+(?!\d))/g, ",")}`;
      },
      formatDate(dateString) {
        if (!dateString) return 'N/A';
        const date = new Date(dateString);
        const options = { year: 'numeric', month: 'short', day: 'numeric' };
        return date.toLocaleDateString('en-US', options);
      },
    },
  };
  </script>
  
  <style scoped>
  .data-table-card {
    background-color: #1a1a1a !important;
    border-radius: 12px;
    box-shadow: 0 4px 15px rgba(0, 0, 0, 0.4);
  }
  .divider-line {
    border-color: #2c2c2c !important;
  }
  .data-list {
    background-color: #1a1a1a !important;
  }
  .customer-list-item {
    border-bottom: 1px solid #2c2c2c;
    transition: background-color 0.3s;
  }
  .customer-list-item:hover {
    background-color: #2c2c2c !important;
  }
  .dashboard-container {
  background-color: #121212 !important;
  min-height: 100vh;
}
  </style>