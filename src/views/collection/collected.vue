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
                    <div v-if="payment.status === 'Collected'" class="primary--text text-subtitle-2 font-weight-bold mt-1" @click="posted(payment)">
                      <v-btn small color="success">Posted</v-btn>
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

    <v-snackbar v-model="snackbar.show" :timeout="snackbar.timeout" :color="snackbar.color" bottom>
      {{ snackbar.message }}
    </v-snackbar>
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
    posted(payment) {
      if (confirm(`Are you sure you want to mark this payment as Posted?`)) {
        axios.post(`http://10.10.10.40:8999/api/collection/payment/posted`, {
          mapid: payment.MapID
        })
        .then((response) => {
          this.snackbar.message = 'Payment status updated to Posted successfully!';
          this.snackbar.color = 'success';
          this.snackbar.show = true;
          // Refresh the payments list
          this.fetchCollectedPayments();
        })
        .catch((error) => {
          console.error('Error updating payment status:', error);
          this.snackbar.message = 'Error updating payment status';
          this.snackbar.color = 'error';
          this.snackbar.show = true;
        });
      }
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