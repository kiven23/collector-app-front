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
                    <div class="primary--text text-subtitle-2 font-weight-bold" @click="getPayment(payment.MapID)">
                      <v-btn small>Receipt</v-btn>
                    </div>
                    <div class="primary--text text-subtitle-2 font-weight-bold mt-1" @click="openMapModal(payment)">
                      <v-btn small>Map</v-btn>
                    </div>
                    <div class="primary--text text-subtitle-2 font-weight-bold mt-2">
                      {{ formatCurrency(payment.CollectedAmount) }}
                    </div>
                    <div class="grey--text text-caption">Amount Paid</div>
                  </div>
                  <v-chip :color="getStatusColor(payment.status)" dark small>
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
    
    <!-- Receipt Modal -->
    <v-dialog v-model="dialogPrintreceipt" max-width="500px">
      <v-card class="receipt-card">
        <v-card-title class="receipt-header justify-center py-4">
          <div class="d-flex flex-column align-center">
            <span class="headline font-weight-bold">OFFICIAL RECEIPT</span>
          </div>
        </v-card-title>

        <v-divider></v-divider>

        <v-card-text class="receipt-body py-6">
          <div class="text-center mb-5">
            <span class="display-1 font-weight-bold primary--text">
              {{ formatCurrency(PaymentReceipt.CollectedAmount) }}
            </span>
          </div>

          <v-list dense class="receipt-details">
            <v-list-item>
              <v-list-item-icon class="mr-2">
                <v-icon small color="primary">mdi-receipt</v-icon>
              </v-list-item-icon>
              <v-list-item-content>
                <v-list-item-title class="font-weight-medium">Receipt No:</v-list-item-title>
              </v-list-item-content>
              <v-list-item-content class="text-right">#{{ PaymentReceipt.ornumber }}</v-list-item-content>
            </v-list-item>

            <v-list-item>
              <v-list-item-icon class="mr-2">
                <v-icon small color="primary">mdi-calendar</v-icon>
              </v-list-item-icon>
              <v-list-item-content>
                <v-list-item-title class="font-weight-medium">Date:</v-list-item-title>
              </v-list-item-content>
              <v-list-item-content class="text-right">{{ formatDate(PaymentReceipt.created_at) }}</v-list-item-content>
            </v-list-item>

            <v-list-item>
              <v-list-item-icon class="mr-2">
                <v-icon small color="primary">mdi-account</v-icon>
              </v-list-item-icon>
              <v-list-item-content>
                <v-list-item-title class="font-weight-medium">Customer:</v-list-item-title>
              </v-list-item-content>
              <v-list-item-content class="text-right">{{ PaymentReceipt.CustomerName }}</v-list-item-content>
            </v-list-item>

            <v-list-item>
              <v-list-item-icon class="mr-2">
                <v-icon small color="primary">mdi-cash-register</v-icon>
              </v-list-item-icon>
              <v-list-item-content>
                <v-list-item-title class="font-weight-medium">Collected By:</v-list-item-title>
              </v-list-item-content>
              <v-list-item-content class="text-right">{{ PaymentReceipt.collectedby }}</v-list-item-content>
            </v-list-item>
          </v-list>

          <div class="dashed-divider my-5"></div>

          <div class="text-center mt-5">
            <p class="font-weight-medium mb-1">Signature:</p>
            <img :src="PaymentReceipt.signature" alt="Signature" height="60" class="signature-image">
          </div>
        </v-card-text>

        <v-card-actions class="justify-space-between pb-4 px-6">
          <v-btn color="secondary" text @click="dialogPrintreceipt = false">Close</v-btn>
          <v-btn color="primary" @click="printReceipt()">Print</v-btn>
        </v-card-actions>

        <v-card-subtitle class="text-center grey--text text--darken-1 pb-4">
          <small>Thank you for your payment.</small><br>
          <small>This is a system-generated receipt and may not require a signature.</small>
        </v-card-subtitle>
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
import axios from "axios";
import L from 'leaflet';
import 'leaflet/dist/leaflet.css';
import { Icon, Marker, LatLng, Polyline } from 'leaflet';

// Fix for default Leaflet icon not appearing
Icon.Default.mergeOptions({
  iconRetinaUrl: require('leaflet/dist/images/marker-icon-2x.png'),
  iconUrl: require('leaflet/dist/images/marker-icon.png'),
  shadowUrl: require('leaflet/dist/images/marker-shadow.png'),
});

export default {
  data() {
    return {
      dialogPrintreceipt: false,
      PaymentReceipt: {},
      payments: [],
      page: 1,
      itemsPerPage: 10,
      totalItems: 0,
      totalPages: 1,
      loading: false,
      map: null,
      mapModal: false,
      routePolyline: null,
      selectedCustomer: {},
      trackerMarker: null,
      trackingInterval: null,
      trackingIndex: 0,
      snackbar: { show: false, message: '', color: 'success', timeout: 3000 },
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
    openMapModal(customer) {
      this.selectedCustomer = { ...customer };
      const gps = [];
      axios.get(`http://10.10.10.40:8999/api/collection/track/get?mapid=${customer.MapID}`)
        .then((res) => {
          res.data.forEach((item) => {
            gps.push([item.Latitude, item.Longitude]);
          });
          this.selectedCustomer.trackingPath = gps;
          this.mapModal = true;
          this.$nextTick(() => {
            this.initMap(gps);
          });
        })
        .catch((error) => {
          console.error('Error fetching GPS data:', error);
          this.snackbar.message = 'Error loading map data';
          this.snackbar.color = 'error';
          this.snackbar.show = true;
        });
    },
    getPayment(id) {
      axios.get(`http://10.10.10.40:8999/api/collection/payment/receipt?mapid=${id}`)
        .then((res) => {
          this.PaymentReceipt = res.data;
          this.dialogPrintreceipt = true;
        })
        .catch((error) => {
          console.error('Error fetching payment receipt:', error);
          this.snackbar.message = 'Error loading receipt';
          this.snackbar.color = 'error';
          this.snackbar.show = true;
        });
    },
    async fetchCollectedPayments() {
      this.loading = true;
      try {
        const response = await axios.get(
          "http://10.10.10.40:8999/api/collection/collected/payments",
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
        this.snackbar.message = 'Error loading payments';
        this.snackbar.color = 'error';
        this.snackbar.show = true;
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
    printReceipt() {
      const printWindow = window.open('', '_blank');
      const receiptContent = `
        <html>
        <head>
          <title>Payment Receipt</title>
          <style>
            body { font-family: Arial, sans-serif; margin: 20px; }
            .receipt-title { text-align: center; font-size: 24px; margin-bottom: 20px; }
            .amount { text-align: center; font-size: 28px; font-weight: bold; margin: 20px 0; }
            .details { margin: 20px 0; }
            .detail-row { display: flex; justify-content: space-between; margin: 10px 0; }
            .signature { text-align: center; margin-top: 20px; }
            .signature img { max-width: 200px; }
            .footer { text-align: center; margin-top: 20px; color: #666; }
            @media print { body { margin: 0; } }
          </style>
        </head>
        <body>
          <div class="receipt-title">Payment Receipt</div>
          <div class="amount">${this.formatCurrency(this.PaymentReceipt.CollectedAmount)}</div>
          <div class="details">
            <div class="detail-row">
              <span>Receipt No:</span>
              <span>#${this.PaymentReceipt.ornumber}</span>
            </div>
            <div class="detail-row">
              <span>Date:</span>
              <span>${this.formatDate(this.PaymentReceipt.created_at)}</span>
            </div>
            <div class="detail-row">
              <span>Customer:</span>
              <span>${this.PaymentReceipt.CustomerName}</span>
            </div>
            <div class="detail-row">
              <span>Collected By:</span>
              <span>${this.PaymentReceipt.collectedby}</span>
            </div>
          </div>
          <div class="signature">
            <p>Signature:</p>
            <img src="${this.PaymentReceipt.signature}" alt="Signature" height="60">
          </div>
          <div class="footer">
            Thank you for your payment.<br>
            This is a system-generated receipt.
          </div>
        </body>
        </html>
      `;
      printWindow.document.write(receiptContent);
      printWindow.document.close();
      printWindow.print();
    },
    initMap(gps) {
      if (this.map) {
        this.map.remove();
      }

      const customerPath = gps || this.selectedCustomer.trackingPath || [];
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
        L.marker(customerPath[0]).addTo(this.map).bindPopup("Start Location").openPopup();
        if (customerPath.length > 1) {
          L.marker(customerPath[customerPath.length - 1]).addTo(this.map).bindPopup("Destination").openPopup();
        }
      }

      this.trackerMarker = L.marker(centerPoint).addTo(this.map)
        .bindPopup("Current location").openPopup();

      if (customerPath.length > 1) {
        this.startTracking();
      }
    },
    startTracking() {
      this.trackingIndex = 0;
      this.trackingInterval = setInterval(() => {
        if (this.trackingIndex < this.selectedCustomer.trackingPath.length - 1) {
          this.trackingIndex++;
          const newPos = this.selectedCustomer.trackingPath[this.trackingIndex];
          this.trackerMarker.setLatLng(newPos).update();
        } else {
          clearInterval(this.trackingInterval);
          this.snackbar.message = "Tracking completed!";
          this.snackbar.color = "info";
          this.snackbar.show = true;
        }
      }, 1000); // Update every 1 second
    },
    closeMapModal() {
      if (this.trackingInterval) {
        clearInterval(this.trackingInterval);
        this.trackingInterval = null;
      }
      if (this.map) {
        this.map.remove();
        this.map = null;
      }
      this.mapModal = false;
      this.selectedCustomer = {};
    },
  },
  beforeDestroy() {
    this.closeMapModal();
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
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}
.receipt-header {
  background-color: #f5f5f5;
}
.receipt-body .v-list-item {
  padding-left: 0;
  padding-right: 0;
}
.receipt-details .v-list-item__title {
  font-size: 0.95rem;
}
.dashed-divider {
  border-bottom: 1px dashed #ccc;
  width: 80%;
  margin: 0 auto;
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
</style>