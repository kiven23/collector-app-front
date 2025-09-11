<template>
  <v-container fluid class="pa-6 dashboard-container">
    <div class="d-flex align-center mb-6">
      <!-- <v-avatar class="user-avatar mr-4">
        <v-img src="https://via.placeholder.com/150/1A1A1A/FFFFFF?text=JD"></v-img>
      </v-avatar> -->
      <!-- <div>
        <h1 class="text-h4 font-weight-bold white--text">
          Hello, Mike!
        </h1>
        <p class="text-subtitle-1 primary--text mb-0">
          Welcome to your Dashboard.
        </p>
      </div> -->
    </div>

    <v-row>
      <v-col cols="12" sm="6" md="4" v-for="(kpi, index) in kpis" :key="index">
        <v-card class="kpi-card pa-4 d-flex align-center">
          <v-icon size="48" class="mr-4 primary--text">{{ kpi.icon }}</v-icon>
          <div>
            <div class="text-h6 font-weight-bold white--text">{{ kpi.value }}</div>
            <div class="text-caption grey--text text--lighten-2">{{ kpi.title }}</div>
          </div>
        </v-card>
      </v-col>
    </v-row>
  
    <v-row>
      <v-col cols="12" md="6">
        <v-card class="data-table-card mt-6">
          <v-card-title class="white--text">
            <v-icon class="mr-2">mdi-calendar-today</v-icon>
            Today's Collection Schedule
          </v-card-title>
        
          <v-divider class="divider-line"></v-divider>
        
          <v-list dark class="pa-0 data-list">
            <div v-if="scheduledCustomersToday.length === 0" class="pa-4 text-center grey--text">
              No collections scheduled for today.
            </div>
            <v-list-item v-for="(customer, index) in scheduledCustomersToday" :key="index" class="customer-list-item">
              <v-list-item-content>
                <div class="d-flex flex-column flex-md-row justify-space-between align-start align-md-center">
                  <div class="d-flex flex-column mb-2 mb-md-0">
                    <div class="white--text font-weight-bold text-subtitle-1">{{ customer.CardName }}</div>
                    <div class="grey--text text-caption">{{ customer.CardCode }} - {{ customer.Branch }}</div>
                  </div>
                  <div class="d-flex flex-row align-center">
                    <div class="text-right mr-4">
                      <div class="primary--text text-subtitle-2 font-weight-bold">{{ formatCurrency(customer.OverDueAmt) }}</div>
                      <div class="grey--text text-caption">Due Today</div>
                    </div>
                  </div>
                </div>
              </v-list-item-content>
            
              <v-list-item-action>
                <v-btn
                  icon
                  color="primary"
                  class="mr-2"
                  @click="onArrived(customer)"
                  aria-label="Mark as Arrived"
                >
                  <v-icon>mdi-map-marker-check</v-icon>
                </v-btn>
              </v-list-item-action>

              <v-list-item-action v-if="customer.arrived == 1">
                <v-btn
                  icon
                  color="primary"
                  class="mr-2"
                  @click="openMapModal(customer)"
                  aria-label="View Map"
                >
                  <v-icon>mdi-map</v-icon>
                </v-btn>
              </v-list-item-action>

              <v-list-item-action>
                <v-btn
                   v-if="customer.arrived == 1"

                  icon
                  color="primary"
                  @click="openPaymentModal(customer)"
                  aria-label="Create Payment"
                >
                  <v-icon>mdi-cash-plus</v-icon>
                </v-btn>
              </v-list-item-action>
            </v-list-item>
          </v-list>
        </v-card>
      </v-col>
    
      <v-col cols="12" md="6">
        <v-card class="data-table-card mt-6" style="min-height: 250px;">
          <v-card-title class="white--text">
            <v-icon class="mr-2">mdi-chart-pie</v-icon>
            Collection Status Distribution
          </v-card-title>
          <v-divider class="divider-line"></v-divider>
          <div class="pa-4">
            <canvas ref="statusChart" style="max-height: 300px;"></canvas>
          </div>
        </v-card>
      </v-col>
    </v-row>

    <v-row>
      <v-col cols="12" md="6">
        <v-card class="data-table-card mt-6" style="min-height: 250px;">
          <v-card-title class="white--text">
            <v-icon class="mr-2">mdi-chart-line</v-icon>
            Monthly Collection Trends
          </v-card-title>
          <v-divider class="divider-line"></v-divider>
          <div class="pa-4">
            <canvas ref="monthlyChart" style="max-height: 300px;"></canvas>
          </div>
        </v-card>
      </v-col>

      <v-col cols="12" md="6">
        <v-card class="data-table-card mt-6" style="min-height: 250px;">
          <v-card-title class="white--text">
            <v-icon class="mr-2">mdi-chart-bar</v-icon>
            Top Overdue Accounts
          </v-card-title>
          <v-divider class="divider-line"></v-divider>
          <div class="pa-4">
            <canvas ref="overdueChart" style="max-height: 300px;"></canvas>
          </div>
        </v-card>
      </v-col>
    </v-row>

    <v-row>
      <v-col cols="12">
        <v-card class="data-table-card mt-6" style="min-height: 250px;">
          <v-card-title class="white--text">
            <v-icon class="mr-2">mdi-target</v-icon>
            Branch Collection Efficiency
          </v-card-title>
          <v-divider class="divider-line"></v-divider>
          <div class="pa-4">
            <div v-if="branchEfficiency.length === 0" class="text-center grey--text">
              No branch data available
            </div>
            <div v-else v-for="branch in branchEfficiency" :key="branch.Branch" class="mb-4">
              <div class="d-flex justify-space-between align-center mb-1">
                <span class="white--text text-subtitle-2">{{ branch.Branch }}</span>
                <span class="primary--text text-subtitle-2">{{ branch.efficiency }}%</span>
              </div>
              <v-progress-linear
                :value="branch.efficiency"
                color="primary"
                height="8"
                rounded
                background-color="#2c2c2c"
              ></v-progress-linear>
              <div class="text-caption grey--text mt-1">
                {{ branch.collected_count }}/{{ branch.total_accounts }} accounts collected
              </div>
            </div>
          </div>
        </v-card>
      </v-col>
    </v-row>

    <v-card class="data-table-card mt-6">
      <v-card-title class="white--text">
        <v-icon class="mr-2">mdi-account-multiple</v-icon>
        Customer Accounts
        <v-spacer></v-spacer>
        <v-text-field
          v-model="search"
          append-icon="mdi-magnify"
          label="Search"
          single-line
          hide-details
          dark
          class="data-search-input"
        ></v-text-field>
        <v-btn
          v-if="selectedCustomers.length > 0"
          @click="scheduleSelected"
          class="ml-4 primary-button"
          color="primary"
          dark
        >
          <v-icon left>mdi-calendar-plus</v-icon>
          Schedule ({{ selectedCustomers.length }})
        </v-btn>
      </v-card-title>
    
      <v-divider class="divider-line"></v-divider>
    
      <v-list dark class="pa-0 data-list">
        <v-list-item v-for="customer in paginatedCustomers" :key="customer.id" class="customer-list-item">
          <v-list-item-content>
            <div class="d-flex flex-column flex-md-row justify-space-between align-start align-md-center">
              <v-checkbox
                v-model="selectedCustomers"
                :value="customer.id"
                color="primary"
                hide-details
                class="mt-0 pt-0 custom-checkbox"
                :disabled="customer.status === 'Collected' || customer.status === 'Posted'"
              ></v-checkbox>
              <div class="d-flex flex-column mb-2 mb-md-0 ml-md-2">
                <div class="white--text font-weight-bold text-subtitle-1">{{ customer.CardName }}</div>
                <div class="grey--text text-caption">{{ customer.CardCode }} - {{ customer.Branch }}</div>
              </div>
              <div class="d-flex flex-row align-center">
                <div class="text-right mr-4">
                  <div class="primary--text text-subtitle-2 font-weight-bold">{{ formatCurrency(customer.OverDueAmt) }}</div>
                  <div class="grey--text text-caption">Overdue Amount</div>
                </div>
                <v-chip :color="getStatusColor(customer.status)" dark>{{ customer.status }}</v-chip>
              </div>
            </div>
          </v-list-item-content>
        </v-list-item>
        <div v-if="!customers.length" class="pa-4 text-center grey--text">
          No customers found.
        </div>
      </v-list>
      <v-pagination
        v-model="page"
        :length="totalPages"
        :total-visible="7"
        color="primary"
        class="mt-4"
      ></v-pagination>
    </v-card>

    <v-dialog v-model="paymentModal" max-width="500px" dark>
      <v-card class="payment-modal-card">
        <v-card-title class="headline white--text">
          Create Payment for {{ selectedCustomer.CardName }}
        </v-card-title>
        <v-divider class="divider-line"></v-divider>
        <v-card-text>
          <div class="my-4">
            <div class="text-subtitle-1 white--text">Overdue Amount: <strong class="primary--text">{{ formatCurrency(selectedCustomer.OverDueAmt) }}</strong></div>
          </div>
        
          <v-text-field
            v-model.number="paymentAmount"
            label="Amount Paid"
            type="number"
            prepend-icon="mdi-currency-usd"
            dark
            class="mb-4"
          ></v-text-field>

          <v-row align="center" class="my-4">
            <v-col cols="6">
              <div class="text-subtitle-1 white--text">Digital Signature:</div>
            </v-col>
            <v-col cols="6" class="text-right">
              <v-btn
                @click="clearSignature"
                text
                color="primary"
                class="text-capitalize"
              >
                <v-icon left>mdi-eraser-variant</v-icon>
                Clear
              </v-btn>
            </v-col>
          </v-row>
        
          <div class="signature-pad pa-4 rounded">
            <canvas
              ref="signatureCanvas"
              width="400"
              height="150"
              class="signature-canvas"
              @mousedown="startDrawing"
              @mousemove="draw"
              @mouseup="stopDrawing"
              @mouseleave="stopDrawing"
              @touchstart.prevent="startDrawingTouch"
              @touchmove.prevent="drawTouch"
              @touchend.prevent="stopDrawing"
            ></canvas>
          </div>
        
        </v-card-text>
        <v-card-actions class="justify-end pa-4">
          <v-btn color="grey darken-2" text @click="paymentModal = false">
            Cancel
          </v-btn>
          <v-btn color="primary" @click="submitPayment">
            <v-icon left>mdi-check</v-icon>
            Submit Payment
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  
    <v-dialog v-model="mapModal" max-width="800px" dark @click:outside="closeMapModal">
      <v-card class="map-modal-card">
        <v-card-title class="headline white--text">
          Live Location Tracking
          <v-spacer></v-spacer>
          <v-btn icon @click="closeMapModal" aria-label="Close Map">
            <v-icon>mdi-close</v-icon>
          </v-btn>
        </v-card-title>
        <v-divider class="divider-line"></v-divider>
        <v-card-text>
          <div id="map-container" style="height: 500px; width: 100%;"></div>
        </v-card-text>
      </v-card>
    </v-dialog>

    <v-snackbar v-model="snackbar.show" :timeout="snackbar.timeout" :color="snackbar.color" bottom>
      {{ snackbar.message }}
    </v-snackbar>

    <!-- Receipt Modal -->
    <v-dialog v-model="dialogPrintreceipt" max-width="500px">
      <v-card class="receipt-card">
        <v-card-title class="receipt-header d-flex flex-column align-center py-5">
          <img src="/logo-addessa.svg" alt="Addessa Corporation" class="logo">
          <span class="text-h5 font-weight-black">COLLECTION RECEIPT</span>
          <span class="text-caption mt-1">The Way To Comfort Living</span>
        </v-card-title>

        <v-divider></v-divider>

        <v-card-text class="receipt-body px-8 py-6">
          <div class="text-center my-4">
            <span class="text-h4 font-weight-bold primary--text">
              {{ formatCurrency(PaymentReceipt.CollectedAmount) }}
            </span>
            <p class="text-subtitle-2 mt-2 grey--text text--darken-1">Amount Paid</p>
          </div>

          <v-divider class="mb-4"></v-divider>

          <v-list dense class="receipt-details">
            <v-list-item>
              <v-list-item-content>
                <v-list-item-title class="font-weight-medium">Receipt No:</v-list-item-title>
              </v-list-item-content>
              <v-list-item-content class="text-right">
                <span class="font-weight-bold">{{ PaymentReceipt.ornumber || '638909' }}</span>
              </v-list-item-content>
            </v-list-item>
            
            <v-list-item>
              <v-list-item-content>
                <v-list-item-title class="font-weight-medium">Payment Date:</v-list-item-title>
              </v-list-item-content>
              <v-list-item-content class="text-right">{{ formatDate(PaymentReceipt.created_at) || '04-10-2024' }}</v-list-item-content>
            </v-list-item>

            <v-list-item>
              <v-list-item-content>
                <v-list-item-title class="font-weight-medium">Received From:</v-list-item-title>
              </v-list-item-content>
              <v-list-item-content class="text-right">{{ PaymentReceipt.CustomerName || 'Olga Garcia P.' }}</v-list-item-content>
            </v-list-item>

            <v-list-item>
              <v-list-item-content>
                <v-list-item-title class="font-weight-medium">Collected By:</v-list-item-title>
              </v-list-item-content>
              <v-list-item-content class="text-right">{{ PaymentReceipt.collectedby || 'N/A' }}</v-list-item-content>
            </v-list-item>
          </v-list>
          
          <v-divider class="my-4"></v-divider>

          <div class="particulars-section">
            <v-simple-table dense class="mb-4">
              <template v-slot:default>
                <thead>
                  <tr>
                    <th class="text-left font-weight-bold">PARTICULARS</th>
                    <th class="text-right font-weight-bold">AMOUNT</th>
                  </tr>
                </thead>
                <tbody>
                  <tr>
                    <td>{{ PaymentReceipt.particulars || '---' }}</td>
                    <td class="text-right">{{ formatCurrency(PaymentReceipt.CollectedAmount) || '₱ 2,749.00' }}</td>
                  </tr>
                  <tr>
                    <td class="font-weight-bold">TOTAL</td>
                    <td class="text-right font-weight-bold">
                      {{ formatCurrency(PaymentReceipt.CollectedAmount) || '₱ 2,749.00' }}
                    </td>
                  </tr>
                </tbody>
              </template>
            </v-simple-table>

            <div class="d-flex justify-space-between text-caption grey--text text--darken-2">
                <span>Payment Method: CASH</span>
                <span>Ref No: {{ PaymentReceipt.ornumber || '638909' }}</span>
            </div>
          </div>
          
          <div class="text-center mt-8">
            <p class="text-caption mb-1">Authorized Signature</p>
            <v-img :src="PaymentReceipt.signature" alt="Signature" height="60" contain class="signature-image d-inline-block"></v-img>
            <v-divider class="my-2"></v-divider>
            <span class="text-caption red--text text--darken-2 d-block">*THIS DOCUMENT IS NOT VALID FOR CLAIM OF INPUT TAX*</span>
          </div>
        </v-card-text>

        <v-card-actions class="justify-space-between pa-6">
          <v-btn color="secondary" text @click="dialogPrintreceipt = false">Cancel</v-btn>
          <v-btn color="primary" @click="printReceipt()">Print</v-btn>
        </v-card-actions>
        
        <v-card-subtitle class="text-center text-caption grey--text text--darken-1 pt-0 pb-4">
          Thank you for your payment.
        </v-card-subtitle>
      </v-card>
    </v-dialog>
  </v-container>
</template>

<script>
import L from 'leaflet';
import 'leaflet/dist/leaflet.css';
import { Icon, Marker, LatLng, Polyline } from 'leaflet';
import axios from "axios";
import Chart from 'chart.js';

// Fix for default Leaflet icon not appearing
// Fix for default Leaflet icon not appearing in Webpack/Vue CLI
Icon.Default.mergeOptions({
  iconRetinaUrl: require('leaflet/dist/images/marker-icon-2x.png'),
  iconUrl: require('leaflet/dist/images/marker-icon.png'),
  shadowUrl: require('leaflet/dist/images/marker-shadow.png'),
});

export default {
  data() {
    const today = new Date();
    const formattedToday = `${today.getFullYear()}-${String(today.getMonth() + 1).padStart(2, '0')}-${String(today.getDate()).padStart(2, '0')}`;
  
    return {
      watchIds: {},
      yourCompanyName: 'Addessa Corporation',
      dialogPrintreceipt: false,
      isDrawing: false,
      search: '',
      PaymentReceipt: [],
      customers: [
        // { cardcode: 'C001', cardname: 'Juan Dela Cruz', branch: 'Main', overdueAmount: 5000.00, status: 'Scheduled', collectionDate: null, arrived: false, trackingPath: [[15.975855447263744,120.5685852],[15.975811,120.5694385],[15.979233,120.5697085],[15.979403,120.5701215],[15.981394688604986, 120.57881989571538],[15.981791481891495, 120.57954220097754],[15.983364476115161, 120.58036769265361]] },
        // { cardcode: 'C002', cardname: 'Maria Santos', branch: 'North', overdueAmount: 0.00, status: 'Collected', collectionDate: null, arrived: false, trackingPath: [] },
        // { cardcode: 'C003', cardname: 'Pedro Reyes', branch: 'South', overdueAmount: 12500.50, status: 'Posted', collectionDate: null, arrived: false, trackingPath: [] },
        // { cardcode: 'C004', cardname: 'Anna Lim', branch: 'Main', overdueAmount: 300.00, status: 'Pending', collectionDate: null, arrived: false, trackingPath: [] },
        // { cardcode: 'C005', cardname: 'Crispin Basco', branch: 'North', overdueAmount: 0.00, status: 'Collected', collectionDate: null, arrived: false, trackingPath: [] },
        // { cardcode: 'C006', cardname: 'Elisa Cruz', branch: 'South', overdueAmount: 750.75, status: 'Posted', collectionDate: null, arrived: false, trackingPath: [] },
        // { cardcode: 'C007', cardname: 'Robert Go', branch: 'Main', overdueAmount: 2000.00, status: 'Pending', collectionDate: null, arrived: false, trackingPath: [] },
      ],
      kpis: [],
      formattedToday: formattedToday,
      selectedCustomers: [],
      snackbar: { show: false, message: '', color: 'success', timeout: 3000 },
      paymentModal: false,
      mapModal: false,
      selectedCustomer: {},
      paymentAmount: null,
      ctx: null,
      map: null,
      trackerMarker: null,
      trackingInterval: null,
      trackingIndex: 0,
      routePolyline: null,
      page: 1,
      itemsPerPage: 10,
      totalItems: 0,
      totalPages: 1,
      searchTimeout: null,
      scheduledCustomers: [],
      // Chart data
      statusChart: null,
      monthlyChart: null,
      overdueChart: null,
      branchEfficiency: [],
      chartData: {
        statusDistribution: [],
        monthlyTrends: [],
        topOverdue: []
      },
    };
  },
  computed: {
    filteredCustomers() {
      // Server-side search and pagination, so just return customers
      return this.customers.filter(customer =>
        customer.status !== 'Scheduled' &&
        customer.status !== 'Collected' &&
        customer.status !== 'Posted'
      );
    },
    scheduledCustomersToday() {
      return this.scheduledCustomers;
    },
    paginatedCustomers() {
      // Server-side pagination, so customers array is already paginated
      return this.filteredCustomers;
    },
  },
  watch: {
    paymentModal(val) {
      if (val) {
        this.$nextTick(() => {
          const canvas = this.$refs.signatureCanvas;
          if (canvas) {
            this.ctx = canvas.getContext("2d");
            this.ctx.strokeStyle = "#ffffff";
            this.ctx.lineWidth = 2;
            this.ctx.lineCap = "round";
          }
        });
      }
    },
    mapModal(val) {
      if (val) {
        this.$nextTick(() => {
          this.initMap();
        });
      }
    },
    page() {
      this.fetchCustomers();
    },
    search() {
      // Debounce search to avoid too many API calls
      clearTimeout(this.searchTimeout);
      this.searchTimeout = setTimeout(() => {
        this.page = 1;
        this.fetchCustomers();
      }, 300);
    },
  },
  mounted() {
      this.fetchCustomers();
      this.fetchScheduledCustomers();
      this.fetchKpis();
      this.fetchChartData();
    },
  methods: {
    fetchCustomers() {
      axios.get('https://collector-api.addessa.com/api/collection/schedule/index', {
        params: {
          page: this.page,
          per_page: this.itemsPerPage,
          search: this.search
        }
      })
      .then(response => {
        console.log('response from backend:', response.data);
        this.customers = response.data.data;
        this.totalItems = response.data.total;
        this.totalPages = response.data.last_page;
      })
      .catch(error => {
        console.error('error from backend:', error);
      });
    },
    fetchScheduledCustomers() {
      axios.get('https://collector-api.addessa.com/api/collection/scheduled/today')
      .then(response => {
        console.log('scheduled customers response:', response.data);
        this.scheduledCustomers = response.data.data;
      })
      .catch(error => {
        console.error('error fetching scheduled customers:', error);
        this.scheduledCustomers = [];
      });
    },
    fetchKpis() {
      axios.get('https://collector-api.addessa.com/api/collection/kpis')
      .then(response => {
        console.log('KPIs response:', response.data);
        this.kpis = [
          { icon: 'mdi-chart-bar', title: 'Total Accounts', value: response.data.total_accounts.toLocaleString() },
          { icon: 'mdi-alert-circle', title: 'Overdue Accounts', value: response.data.overdue_accounts.toLocaleString() },
          { icon: 'mdi-currency-usd', title: 'Amount to Collect', value: this.formatCurrency(response.data.amount_to_collect) },
        ];
      })
      .catch(error => {
        console.error('error fetching KPIs:', error);
        // Fallback to default values if API fails
        this.kpis = [
          { icon: 'mdi-chart-bar', title: 'Total Accounts', value: '0' },
          { icon: 'mdi-alert-circle', title: 'Overdue Accounts', value: '0' },
          { icon: 'mdi-currency-usd', title: 'Amount to Collect', value: '₱ 0.00' },
        ];
      });
    },
    fetchChartData() {
      axios.get('https://collector-api.addessa.com/api/collection/chart-data')
      .then(response => {
        console.log('Chart data response:', response.data);
        this.chartData = response.data;
        this.branchEfficiency = response.data.branch_efficiency || [];
        this.$nextTick(() => {
          this.createStatusChart();
          this.createMonthlyChart();
          this.createOverdueChart();
        });
      })
      .catch(error => {
        console.error('error fetching chart data:', error);
        // Set default empty data
        this.chartData = {
          status_distribution: [],
          monthly_trends: [],
          top_overdue: []
        };
        this.branchEfficiency = [];
      });
    },
    createStatusChart() {
      const ctx = this.$refs.statusChart;
      if (!ctx) return;

      const data = this.chartData.status_distribution || [];
      const labels = data.map(item => item.status);
      const values = data.map(item => item.count);

      if (this.statusChart) {
        this.statusChart.destroy();
      }

      this.statusChart = new Chart(ctx, {
        type: 'pie',
        data: {
          labels: labels,
          datasets: [{
            data: values,
            backgroundColor: [
              '#42a5f5', // Scheduled - Blue
              '#e53935', // Overdue - Red
              '#43a047', // Collected - Green
              '#ffb300', // Pending - Yellow
              '#673ab7', // Posted - Purple
              '#616161'  // Default - Grey
            ],
            borderWidth: 2,
            borderColor: '#1a1a1a'
          }]
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
          plugins: {
            legend: {
              labels: {
                color: '#ffffff',
                font: {
                  size: 12
                }
              }
            }
          }
        }
      });
    },
    createMonthlyChart() {
      const ctx = this.$refs.monthlyChart;
      if (!ctx) return;

      const data = this.chartData.monthly_trends || [];
      const labels = data.map(item => {
        const date = new Date(item.month + '-01');
        return date.toLocaleDateString('en-US', { month: 'short', year: 'numeric' });
      });
      const values = data.map(item => parseFloat(item.total));

      if (this.monthlyChart) {
        this.monthlyChart.destroy();
      }

      this.monthlyChart = new Chart(ctx, {
        type: 'line',
        data: {
          labels: labels,
          datasets: [{
            label: 'Collection Amount',
            data: values,
            borderColor: '#42a5f5',
            backgroundColor: 'rgba(66, 165, 245, 0.1)',
            borderWidth: 3,
            fill: true,
            tension: 0.4
          }]
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
          scales: {
            y: {
              beginAtZero: true,
              ticks: {
                color: '#ffffff',
                callback: (value) => this.formatCurrency(value)
              },
              grid: {
                color: '#2c2c2c'
              }
            },
            x: {
              ticks: {
                color: '#ffffff'
              },
              grid: {
                color: '#2c2c2c'
              }
            }
          },
          plugins: {
            legend: {
              labels: {
                color: '#ffffff'
              }
            }
          }
        }
      });
    },
    formatDate(dateString) {
      if (!dateString) return "N/A";
      const date = new Date(dateString);
      return date.toLocaleDateString("en-US", { year: "numeric", month: "short", day: "numeric" });
    },
    createOverdueChart() {
      const ctx = this.$refs.overdueChart;
      if (!ctx) return;

      const data = this.chartData.top_overdue || [];
      const labels = data.map(item => item.CardName.length > 15 ? item.CardName.substring(0, 15) + '...' : item.CardName);
      const values = data.map(item => parseFloat(item.OverDueAmt));

      if (this.overdueChart) {
        this.overdueChart.destroy();
      }

      this.overdueChart = new Chart(ctx, {
        type: 'bar',
        data: {
          labels: labels,
          datasets: [{
            label: 'Overdue Amount',
            data: values,
            backgroundColor: '#e53935',
            borderColor: '#b71c1c',
            borderWidth: 1
          }]
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
          scales: {
            y: {
              beginAtZero: true,
              ticks: {
                color: '#ffffff',
                callback: (value) => this.formatCurrency(value)
              },
              grid: {
                color: '#2c2c2c'
              }
            },
            x: {
              ticks: {
                color: '#ffffff',
                maxRotation: 45,
                minRotation: 45
              },
              grid: {
                color: '#2c2c2c'
              }
            }
          },
          plugins: {
            legend: {
              labels: {
                color: '#ffffff'
              }
            }
          }
        }
      });
    },
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
    const num = Number(amount); // convert string to number
    if (isNaN(num)) return '₱ 0.00';
    return `₱ ${num.toFixed(2).replace(/\B(?=(\d{3})+(?!\d))/g, ",")}`;
  },
    scheduleSelected() {
      if (this.selectedCustomers.length === 0) return;
    
      const scheduledCount = this.selectedCustomers.length;
    
      this.selectedCustomers.forEach(id => {
        const customer = this.customers.find(c => c.id === id);
        if (customer) {
          customer.status = 'Scheduled';
           
        }
      });
       axios.post('https://collector-api.addessa.com/api/collection/schedule/set', {
            data: this.selectedCustomers
          })
          .then(response => {
            // Refresh scheduled customers data after successful scheduling
            this.fetchScheduledCustomers();
          })
      this.selectedCustomers = [];
      this.snackbar.message = `${scheduledCount} customer(s) have been scheduled for today.`;
      this.snackbar.show = true;
    },
    onArrived(customer) {
      console.log(customer)
      const mapid = customer.MapID;

      // prevent duplicate watch
      if (this.watchIds[mapid]) {
        console.log(`Already tracking ${mapid}`);
        return;
      }

      const watchId = navigator.geolocation.watchPosition(
        (position) => {
          const lat = position.coords.latitude;
          const lng = position.coords.longitude;

          axios.post('https://collector-api.addessa.com/api/collection/arrived/set', {
            data: {
              mapid: mapid,
              Latitude: lat,
              Longitude: lng,
            },
          })
          .then(response => {
            console.log(`Location updated for ${mapid}:`, response.data);
            this.fetchScheduledCustomers();
          });
        },
        (error) => {
          console.error(`GPS error for ${mapid}:`, error);
        },
        {
          enableHighAccuracy: true,
          maximumAge: 0,
          timeout: 10000,
        }
      );

      // save watchId per mapid
      this.watchIds[mapid] = watchId;

      //GPS TRACKING END
      this.snackbar.message = `Arrived at ${customer.CardName}'s location.`;
      this.snackbar.color = 'info';
      this.snackbar.show = true;
    },
    stopTracking(mapid) {
      if (this.watchIds[mapid]) {
        navigator.geolocation.clearWatch(this.watchIds[mapid]);
        delete this.watchIds[mapid];
        console.log(`Stopped tracking ${mapid}`);
      }
    },
    openPaymentModal(customer) {
      this.selectedCustomer = { ...customer };
      this.paymentAmount = customer.OverDueAmt;
      this.paymentModal = true;
      this.clearSignature();
    },
  
    submitPayment() {
     
      const signatureData = this.getSignatureImage();
      const isEmpty = !signatureData || signatureData.length < 100;
    
      if (isEmpty) {
        this.snackbar.message = 'Please provide a digital signature.';
        this.snackbar.color = 'warning';
        this.snackbar.show = true;
        return;
      }
      
      const customerIndex = this.scheduledCustomers.findIndex(c => c.CardCode === this.selectedCustomer.CardCode);
    
      if (customerIndex !== -1) {
          this.scheduledCustomers[customerIndex].status = 'Collected';
          this.scheduledCustomers[customerIndex].OverDueAmt = 0;
         
         
          var data;
          data = {
            MapID: this.scheduledCustomers[customerIndex].MapID,
            CardCode: this.scheduledCustomers[customerIndex].CardCode,
            CollectedAmount: this.paymentAmount,
            Signature: signatureData,
            CollectedBy: this.scheduledCustomers[customerIndex].user_id
          }
   
         axios.post('https://collector-api.addessa.com/api/collection/payment/set',data ).then((res)=>{
              console.log(res)
              this.PaymentReceipt = res.data.data
              this.dialogPrintreceipt = true
              this.stopTracking(this.PaymentReceipt.MapID)  
              
              // Refresh both customer lists after payment
              this.fetchCustomers()
              this.fetchScheduledCustomers()
         })
      }
       
      this.paymentModal = false;
      this.snackbar.message = `Payment of ${this.formatCurrency(this.paymentAmount)} successfully collected from ${this.selectedCustomer.CardName}.`;
      this.snackbar.color = 'success';
      this.snackbar.show = true;
    },

    openMapModal(customer) {
      axios.get('https://collector-api.addessa.com/api/collection/track/get?mapid='+customer.MapID ).then((res)=>{
        var gps = [];
        res.data.forEach((item,index)=>{
          gps.push([item.Latitude , item.Longitude])
        })
        this.selectedCustomer.trackingPath =gps
        console.log(gps)
      })
      this.selectedCustomer = customer;
      this.mapModal = true;
    },
    closeMapModal() {
      if (this.trackingInterval) {
        clearInterval(this.trackingInterval);
      }
      if (this.map) {
        this.map.remove();
        this.map = null;
      }
      this.mapModal = false;
    },
  
    initMap() {
      if (this.map) {
        this.map.remove();
      }
      
      const customerPath = this.selectedCustomer.trackingPath;
      const centerPoint = customerPath.length > 0 ? customerPath[0] : [15.968, 120.575];

      this.map = L.map('map-container').setView(centerPoint, 14);
      L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        attribution: 'Map data &copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap contributors</a>',
      }).addTo(this.map);

      // Draw the entire route as a polyline
      if (customerPath.length > 1) {
        this.routePolyline = new L.Polyline(customerPath, {
          color: '#673ab7',
          weight: 5,
          opacity: 0.8
        }).addTo(this.map);
        this.map.fitBounds(this.routePolyline.getBounds());
      }

      // Add start and end markers
      if (customerPath.length > 0) {
        L.marker(customerPath[0], { icon: new Icon.Default({ iconUrl: require('leaflet/dist/images/marker-icon.png'), iconSize: [25, 41], iconAnchor: [12, 41] }) }).addTo(this.map).bindPopup("Start Location").openPopup();
        L.marker(customerPath[customerPath.length - 1], { icon: new Icon.Default({ iconUrl: require('leaflet/dist/images/marker-icon.png'), iconSize: [25, 41], iconAnchor: [12, 41] }) }).addTo(this.map).bindPopup("Destination").openPopup();
      }
      
      this.trackerMarker = L.marker(centerPoint).addTo(this.map)
        .bindPopup("Your current location").openPopup();

      this.startTracking();
    },

    startTracking() {
      this.trackingIndex = 0;
      this.trackingInterval = setInterval(() => {
        if (this.trackingIndex < this.selectedCustomer.trackingPath.length - 1) {
          this.trackingIndex++;
          const newPos = this.selectedCustomer.trackingPath[this.trackingIndex];
          this.trackerMarker.setLatLng(newPos).update();
          // No need for map.panTo() as fitBounds already handles a good view
        } else {
          clearInterval(this.trackingInterval);
          this.snackbar.message = "Tracking completed!";
          this.snackbar.color = "info";
          this.snackbar.show = true;
        }
      }, 1000); // Update every 1 second
    },

    ////DIGITAL SIGNATURE METHODS ////
    startDrawing(e) {
      this.isDrawing = true;
      const rect = this.$refs.signatureCanvas.getBoundingClientRect();
      this.ctx.beginPath();
      this.ctx.moveTo(e.clientX - rect.left, e.clientY - rect.top);
    },
    draw(e) {
      if (!this.isDrawing) return;
      const rect = this.$refs.signatureCanvas.getBoundingClientRect();
      this.ctx.lineTo(e.clientX - rect.left, e.clientY - rect.top);
      this.ctx.stroke();
    },
    stopDrawing() {
      this.isDrawing = false;
      this.ctx.closePath();
    },
    startDrawingTouch(e) {
      const rect = this.$refs.signatureCanvas.getBoundingClientRect();
      const touch = e.touches[0];
      this.isDrawing = true;
      this.ctx.beginPath();
      this.ctx.moveTo(touch.clientX - rect.left, touch.clientY - rect.top);
    },
    drawTouch(e) {
      if (!this.isDrawing) return;
      const rect = this.$refs.signatureCanvas.getBoundingClientRect();
      const touch = e.touches[0];
      this.ctx.lineTo(touch.clientX - rect.left, touch.clientY - rect.top);
      this.ctx.stroke();
    },
    clearSignature() {
      if (this.ctx) {
          const canvas = this.$refs.signatureCanvas;
          this.ctx.clearRect(0, 0, canvas.width, canvas.height);
      }
    },
    getSignatureImage() {
      const canvas = this.$refs.signatureCanvas;
      return canvas.toDataURL("image/jpeg");
    },
    printReceipt() {
      const printWindow = window.open('', '_blank');
      const receiptContent = `
        <html>
          <head>
            <title>Collection Receipt - Addessa</title>
            <style>
              body {
                font-family: 'Roboto', sans-serif;
                margin: 0;
                padding: 20px;
                color: #333;
                background: #fff;
              }
              .receipt-container {
                width: 100%;
                max-width: 450px;
                margin: 0 auto;
                border: 1px solid #ddd;
                padding: 30px;
                box-shadow: 0 0 10px rgba(0,0,0,0.1);
              }
              .header {
                text-align: center;
                margin-bottom: 20px;
              }
              .logo {
                max-width: 150px;
                margin-bottom: 10px;
              }
              .receipt-title {
                font-size: 24px;
                font-weight: bold;
                color: #333;
              }
              .subtitle {
                font-size: 14px;
                color: #777;
              }
              .amount-section {
                text-align: center;
                margin: 20px 0;
              }
              .amount {
                font-size: 32px;
                font-weight: bold;
                color: #1976D2; /* primary--text in Vuetify */
              }
              .label {
                font-size: 12px;
                color: #888;
              }
              .details-list {
                list-style: none;
                padding: 0;
                margin: 0;
              }
              .details-list li {
                display: flex;
                justify-content: space-between;
                padding: 8px 0;
                border-bottom: 1px dashed #eee;
              }
              .details-list li:last-child {
                border-bottom: none;
              }
              .details-list .label-text {
                font-weight: 500;
              }
              .details-list .value-text {
                font-weight: 400;
              }
              .table-section {
                margin-top: 20px;
                border-top: 1px solid #ddd;
                padding-top: 20px;
              }
              table {
                width: 100%;
                border-collapse: collapse;
                font-size: 14px;
              }
              table th, table td {
                padding: 8px 0;
                text-align: left;
              }
              table th:last-child, table td:last-child {
                text-align: right;
              }
              .total-row {
                font-weight: bold;
                border-top: 1px solid #ddd;
              }
              .disclaimer {
                text-align: center;
                font-size: 12px;
                color: #E53935; /* red--text in Vuetify */
                margin-top: 20px;
              }
              .signature-section {
                text-align: center;
                margin-top: 30px;
              }
              .signature-section .label-text {
                font-size: 12px;
                color: #555;
                margin-bottom: 5px;
              }
              .signature-image {
                max-width: 150px;
                height: 60px;
                object-fit: contain;
              }
              .footer-section {
                text-align: center;
                margin-top: 30px;
                font-size: 12px;
                color: #777;
                padding-top: 10px;
                border-top: 1px solid #eee;
              }
              /* Hide buttons and dialog background for printing */
              @media print {
                .no-print { display: none; }
                body {
                  padding: 0 !important;
                  margin: 0 !important;
                }
              }
            </style>
          </head>
          <body>
            <div class="receipt-container">
              <div class="header">
                <img src="/logo-addessa.svg" alt="Addessa Corporation" class="logo">
                <div class="receipt-title">COLLECTION RECEIPT</div>
                <div class="subtitle">The Way To Comfort Living</div>
              </div>

              <hr style="border: none; border-top: 1px solid #ddd; margin: 20px 0;">
              <div class="amount-section">
                <div class="amount">
                  ${this.formatCurrency(this.PaymentReceipt.CollectedAmount)}
                </div>
                <div class="label">Amount Paid</div>
              </div>
              <hr style="border: none; border-top: 1px solid #ddd; margin: 20px 0;">

              <ul class="details-list">
                <li>
                  <span class="label-text">Receipt No:</span>
                  <span class="value-text">#${this.PaymentReceipt.ornumber || '638909'}</span>
                </li>
                <li>
                  <span class="label-text">Payment Date:</span>
                  <span class="value-text">${this.formatDate(this.PaymentReceipt.created_at) || '04-10-2024'}</span>
                </li>
                <li>
                  <span class="label-text">Received From:</span>
                  <span class="value-text">${this.PaymentReceipt.CustomerName || 'Olga Garcia P.'}</span>
                </li>
                <li>
                  <span class="label-text">Collected By:</span>
                  <span class="value-text">${this.PaymentReceipt.collectedby || 'N/A'}</span>
                </li>
              </ul>

              <div class="table-section">
                <table>
                  <thead>
                    <tr>
                      <th>PARTICULARS</th>
                      <th>AMOUNT</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr>
                      <td>${this.PaymentReceipt.particulars || '---'}</td>
                      <td>${this.formatCurrency(this.PaymentReceipt.CollectedAmount) || '₱ 2,749.00'}</td>
                    </tr>
                    <tr class="total-row">
                      <td>TOTAL</td>
                      <td>${this.formatCurrency(this.PaymentReceipt.CollectedAmount) || '₱ 2,749.00'}</td>
                    </tr>
                  </tbody>
                </table>
              </div>

              <div style="font-size: 12px; color: #777; margin-top: 10px; display: flex; justify-content: space-between;">
                <span>Payment Method: CASH</span>
                <span>Ref No: ${this.PaymentReceipt.ornumber || '638909'}</span>
              </div>

              <div class="signature-section">
                <p class="label-text">Authorized Signature</p>
                <img src="${this.PaymentReceipt.signature}" alt="Signature" class="signature-image">
                <hr style="border: none; border-top: 1px dashed #ddd; margin-top: 20px;">
                <div class="disclaimer">
                  *THIS DOCUMENT IS NOT VALID FOR CLAIM OF INPUT TAX*
                </div>
              </div>

              <div class="footer-section">
                Thank you for your payment.
              </div>
            </div>
          </body>
        </html>
      `;

      printWindow.document.write(receiptContent);
      printWindow.document.close();
      printWindow.print();
      printWindow.onafterprint = () => {
        printWindow.close();
      };
    },
  beforeDestroy() {
    this.closeMapModal();
    if (this.searchTimeout) {
      clearTimeout(this.searchTimeout);
    }
    // Clean up charts
    if (this.statusChart) {
      this.statusChart.destroy();
    }
    if (this.monthlyChart) {
      this.monthlyChart.destroy();
    }
    if (this.overdueChart) {
      this.overdueChart.destroy();
    }
  }
}
};
</script>

<style scoped>

.dashboard-container {
  background-color: #121212 !important;
  min-height: 100vh;
}

.user-avatar {
  border: 2px solid #673ab7;
  box-shadow: 0 0 10px rgba(103, 58, 183, 0.5);
}

.kpi-card {
  background-color: #1a1a1a !important;
  color: #fff;
  border-radius: 12px;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.4), 0 0 10px rgba(103, 58, 183, 0.2);
}

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

.data-search-input.v-text-field {
  max-width: 300px;
}

.primary-button {
  transition: all 0.3s ease-in-out;
  background-color: #673ab7 !important;
  color: white !important;
}

.primary-button:hover {
  box-shadow: 0 6px 20px rgba(103, 58, 183, 0.6);
  transform: translateY(-2px);
}

.payment-modal-card {
  background-color: #1a1a1a !important;
}

.map-modal-card {
  background-color: #1a1a1a !important;
}

.signature-pad {
  background-color: #2c2c2c;
  border: 1px solid #424242;
}

#map-container {
  z-index: 0;
}


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
  max-height: 600px;
  overflow-y: auto;
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
.receipt-card {
  font-family: 'Arial', sans-serif;
  border: 1px solid #000;
  background-color: #fff;
}
.receipt-header {
  background-color: #f5f5f5;
  padding: 10px;
}
.receipt-body {
  padding: 10px;
}
.receipt-details .v-list-item {
  padding-left: 0;
  padding-right: 0;
}
.receipt-details .v-list-item__title {
  font-size: 0.95rem;
}
.v-simple-table {
  width: 100%;
  border: 1px solid #000;
}
.v-simple-table th {
  border-bottom: 1px solid #000;
  padding: 5px;
  font-weight: bold;
}
.v-simple-table td {
  padding: 5px;
  border-bottom: 1px dashed #000;
}
.red--text {
  color: red;
}
.signature-image {
  border-bottom: 1px solid #000;
  padding-bottom: 5px;
}
.map-modal-card {
  background-color: #1a1a1a !important;
}
#map-container {
  border-radius: 8px;
}
.button-custom {
  transition: all 0.3s ease;
  text-transform: none !important;
  font-weight: 500;
}
.button-custom:hover {
  background-color: rgba(255, 255, 255, 0.1);
  transform: translateY(-1px);
}
.button-custom:active {
  transform: translateY(0);
}
.logo {
        max-width: 150px;
        margin-bottom: 10px;
      }
</style>