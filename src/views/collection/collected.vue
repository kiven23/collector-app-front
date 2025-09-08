<template>
  <v-container fluid class="pa-6 dashboard-container">
    <v-card class="data-table-card mt-6">
      <v-card-title class="white--text">
        <v-icon class="mr-2">mdi-history</v-icon>
        Collected and Posted Payments
      </v-card-title>
      <v-card-subtitle class="grey--text text--lighten-2"></v-card-subtitle>

      <v-divider class="divider-line"></v-divider>

      <v-list dark class="pa-0 data-list">
        <!-- Loading State -->
        <div v-if="loading" class="pa-4 text-center grey--text">
          <v-progress-circular indeterminate color="primary"></v-progress-circular>
          <div class="mt-2">Loading payments...</div>
        </div>

        <!-- Empty State -->
        <div v-else-if="filteredPayments.length === 0" class="pa-4 text-center grey--text">
          No collected or posted payments yet.
        </div>

        <!-- Payments List -->
        <div v-else>
          <v-list-item
            v-for="(payment, index) in filteredPayments"
            :key="`payment-${index}`"
            class="customer-list-item"
          >
            <v-list-item-content>
              <div class="d-flex flex-column flex-md-row justify-space-between align-start align-md-center">
                <!-- Customer Info -->
                <div class="d-flex flex-column mb-2 mb-md-0">
                  <div class="white--text font-weight-bold text-subtitle-1">
                    {{ payment.CustomerName || payment.CardName }}
                  </div>
                  <div class="grey--text text-caption">
                    {{ payment.CardCode }} - {{ payment.Branch }}
                  </div>
                  <div class="grey--text text-caption">
                    {{ formatDate(payment.created_at) }}
                  </div>
                </div>

                <!-- Amount + Status -->
                <div class="d-flex flex-row align-center">
                  <div class="text-right mr-4">
                    <div class="primary--text text-subtitle-2 font-weight-bold">
                      {{ formatCurrency(payment.CollectedAmount) }}
                    </div>
                    <div class="grey--text text-caption">Amount Paid</div>
                  </div>
                  <v-chip :color="getStatusColor(payment.status)" dark>
                    {{ payment.status }}
                  </v-chip>
                </div>
              </div>
            </v-list-item-content>
          </v-list-item>
        </div>
      </v-list>

      <!-- Pagination -->
      <v-pagination
        v-model="page"
        :length="totalPages"
        :total-visible="7"
        color="primary"
        class="mt-4"
      ></v-pagination>
    </v-card>
  </v-container>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      payments: [],
      page: 1,
      itemsPerPage: 10,
      totalItems: 0,
      totalPages: 1,
      loading: false,
    };
  },
  computed: {
    filteredPayments() {
      return this.payments || [];
    },
  },
  mounted() {
    this.fetchCollectedPayments();
  },
  watch: {
    page() {
      this.fetchCollectedPayments();
    },
  },
  methods: {
    async fetchCollectedPayments() {
      this.loading = true;
      try {
        const response = await axios.get(
          "http://localhost:8000/api/collection/collected/payments",
          {
            params: {
              page: this.page,
              per_page: this.itemsPerPage,
            },
          }
        );

        this.payments = response.data?.data || [];
        this.totalItems = response.data?.total || 0;
        this.totalPages = response.data?.last_page || 1;
      } catch (error) {
        console.error("Error fetching collected payments:", error.response || error);
        this.payments = [];
        this.totalItems = 0;
        this.totalPages = 1;
      } finally {
        this.loading = false;
      }
    },
    getStatusColor(status) {
      switch (status) {
        case "Scheduled": return "#42a5f5";
        case "Overdue": return "#e53935";
        case "Collected": return "#43a047";
        case "Pending": return "#ffb300";
        case "Posted": return "#673ab7";
        default: return "#616161";
      }
    },
    formatCurrency(amount) {
      const num = parseFloat(amount);
      if (isNaN(num)) return "₱ 0.00";
      return `₱ ${num.toFixed(2).replace(/\B(?=(\d{3})+(?!\d))/g, ",")}`;
    },
    formatDate(dateString) {
      if (!dateString) return "N/A";
      const date = new Date(dateString);
      return date.toLocaleDateString("en-US", { year: "numeric", month: "short", day: "numeric" });
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
