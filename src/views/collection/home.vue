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
                    <div class="white--text font-weight-bold text-subtitle-1">{{ customer.cardname }}</div>
                    <div class="grey--text text-caption">{{ customer.cardcode }} - {{ customer.branch }}</div>
                  </div>
                  <div class="d-flex flex-row align-center">
                    <div class="text-right mr-4">
                      <div class="primary--text text-subtitle-2 font-weight-bold">{{ formatCurrency(customer.overdueAmount) }}</div>
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

              <v-list-item-action v-if="customer.arrived">
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
            <v-icon class="mr-2">mdi-chart-line</v-icon>
            Placeholder for Charts or Reports
          </v-card-title>
          <div class="pa-4 text-center grey--text">
            [Your charts or other data visualizations here]
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
        <v-list-item v-for="(customer, index) in filteredCustomers" :key="index" class="customer-list-item">
          <v-list-item-content>
            <div class="d-flex flex-column flex-md-row justify-space-between align-start align-md-center">
              <v-checkbox
                v-model="selectedCustomers"
                :value="customer.cardcode"
                color="primary"
                hide-details
                class="mt-0 pt-0 custom-checkbox"
                :disabled="customer.status === 'Collected' || customer.status === 'Posted'"
              ></v-checkbox>
              <div class="d-flex flex-column mb-2 mb-md-0 ml-md-2">
                <div class="white--text font-weight-bold text-subtitle-1">{{ customer.cardname }}</div>
                <div class="grey--text text-caption">{{ customer.cardcode }} - {{ customer.branch }}</div>
              </div>
              <div class="d-flex flex-row align-center">
                <div class="text-right mr-4">
                  <div class="primary--text text-subtitle-2 font-weight-bold">{{ formatCurrency(customer.overdueAmount) }}</div>
                  <div class="grey--text text-caption">Overdue Amount</div>
                </div>
                <v-chip :color="getStatusColor(customer.status)" dark>{{ customer.status }}</v-chip>
              </div>
            </div>
          </v-list-item-content>
        </v-list-item>
        <div v-if="!filteredCustomers.length" class="pa-4 text-center grey--text">
          No matching customers found.
        </div>
      </v-list>
    </v-card>

    <v-dialog v-model="paymentModal" max-width="500px" dark>
      <v-card class="payment-modal-card">
        <v-card-title class="headline white--text">
          Create Payment for {{ selectedCustomer.cardname }}
        </v-card-title>
        <v-divider class="divider-line"></v-divider>
        <v-card-text>
          <div class="my-4">
            <div class="text-subtitle-1 white--text">Overdue Amount: <strong class="primary--text">{{ formatCurrency(selectedCustomer.overdueAmount) }}</strong></div>
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
  </v-container>
</template>

<script>
import L from 'leaflet';
import 'leaflet/dist/leaflet.css';
import { Icon, Marker, LatLng, Polyline } from 'leaflet';

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
      isDrawing: false,
      search: '',
      customers: [
        { cardcode: 'C001', cardname: 'Juan Dela Cruz', branch: 'Main', overdueAmount: 5000.00, status: 'Scheduled', collectionDate: null, arrived: false, trackingPath: [[15.975855447263744,120.5685852],[15.975811,120.5694385],[15.979233,120.5697085],[15.979403,120.5701215],[15.981394688604986, 120.57881989571538],[15.981791481891495, 120.57954220097754],[15.983364476115161, 120.58036769265361]] },
        { cardcode: 'C002', cardname: 'Maria Santos', branch: 'North', overdueAmount: 0.00, status: 'Collected', collectionDate: null, arrived: false, trackingPath: [] },
        { cardcode: 'C003', cardname: 'Pedro Reyes', branch: 'South', overdueAmount: 12500.50, status: 'Posted', collectionDate: null, arrived: false, trackingPath: [] },
        { cardcode: 'C004', cardname: 'Anna Lim', branch: 'Main', overdueAmount: 300.00, status: 'Pending', collectionDate: null, arrived: false, trackingPath: [] },
        { cardcode: 'C005', cardname: 'Crispin Basco', branch: 'North', overdueAmount: 0.00, status: 'Collected', collectionDate: null, arrived: false, trackingPath: [] },
        { cardcode: 'C006', cardname: 'Elisa Cruz', branch: 'South', overdueAmount: 750.75, status: 'Posted', collectionDate: null, arrived: false, trackingPath: [] },
        { cardcode: 'C007', cardname: 'Robert Go', branch: 'Main', overdueAmount: 2000.00, status: 'Pending', collectionDate: null, arrived: false, trackingPath: [] },
      ],
      kpis: [
        { icon: 'mdi-chart-bar', title: 'Total Accounts', value: '1,250' },
        { icon: 'mdi-alert-circle', title: 'Overdue Accounts', value: '187' },
        { icon: 'mdi-currency-usd', title: 'Amount to Collect', value: '₱ 2.5M' },
      ],
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
    };
  },
  computed: {
    filteredCustomers() {
      const searchText = this.search.toLowerCase();
      return this.customers.filter(customer => {
        const matchesSearch = Object.values(customer).some(value =>
          String(value).toLowerCase().includes(searchText)
        );
        const isAvailableForScheduling = customer.status !== 'Scheduled' && customer.status !== 'Collected' && customer.status !== 'Posted';
        return matchesSearch && isAvailableForScheduling;
      });
    },
    scheduledCustomersToday() {
      return this.customers.filter(customer => customer.status === 'Scheduled');
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
    scheduleSelected() {
      if (this.selectedCustomers.length === 0) return;
    
      const scheduledCount = this.selectedCustomers.length;
    
      this.selectedCustomers.forEach(cardcode => {
        const customer = this.customers.find(c => c.cardcode === cardcode);
        if (customer) {
          customer.status = 'Scheduled';
        }
      });
    
      this.selectedCustomers = [];
      this.snackbar.message = `${scheduledCount} customer(s) have been scheduled for today.`;
      this.snackbar.show = true;
    },
    onArrived(customer) {
      const customerIndex = this.customers.findIndex(c => c.cardcode === customer.cardcode);
      if (customerIndex !== -1) {
        this.customers[customerIndex].arrived = true;
      }
      this.snackbar.message = `Arrived at ${customer.cardname}'s location.`;
      this.snackbar.color = 'info';
      this.snackbar.show = true;
    },
    openPaymentModal(customer) {
      this.selectedCustomer = { ...customer };
      this.paymentAmount = customer.overdueAmount;
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
    
      const customerIndex = this.customers.findIndex(c => c.cardcode === this.selectedCustomer.cardcode);
      if (customerIndex !== -1) {
        this.customers[customerIndex].status = 'Collected';
        this.customers[customerIndex].overdueAmount = 0;
      }
    
      this.paymentModal = false;
      this.snackbar.message = `Payment of ${this.formatCurrency(this.paymentAmount)} successfully collected from ${this.selectedCustomer.cardname}.`;
      this.snackbar.color = 'success';
      this.snackbar.show = true;
    },

    openMapModal(customer) {
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
  },
  beforeDestroy() {
    this.closeMapModal();
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
</style>