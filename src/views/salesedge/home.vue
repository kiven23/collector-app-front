<template>
  <v-container fluid class="pa-0">
    <v-tabs v-model="tab" class="sales-edge-tabs" align-with-title>
      <v-tab key="dashboard">
        <v-icon left>mdi-view-dashboard</v-icon>Dashboard
      </v-tab>
      <v-tab key="employees">
        <v-icon left>mdi-account-group</v-icon>Employees
      </v-tab>
      
      <v-tab key="approved"  >
        <v-icon left>mdi-download</v-icon>Approved Product Bonus
      </v-tab>
      <v-tab key="sales" v-if="sales_upload_product" >
        <v-icon left>mdi-shopping</v-icon>Product Maintenance
      </v-tab>
    </v-tabs>

    <v-tabs-items v-model="tab">
      <v-tab-item key="dashboard" class="pa-4">
        <v-card class="elevation-2 pa-4 mb-6 dashboard-card">
          <h2 class="text-h5 font-weight-bold primary--text mb-2">📊 Performance Overview</h2>
          <p class="subtitle-1 grey--text text--darken-1">Visualize key sales metrics and analytics.</p>
        </v-card>
        <v-row justify="center">
          <v-col cols="12" md="10" lg="9">
            <v-card class="elevation-5 chart-card">
              <v-img
                :src="imageUrl"
                alt="Sales Performance Chart"
                class="rounded-lg chart-image"
                cover
                min-height="400px"
              ></v-img>
              <v-overlay absolute :value="!imageUrl" color="grey lighten-3" opacity="0.8">
                <v-progress-circular indeterminate color="primary"></v-progress-circular>
                <div class="mt-3 text-caption primary--text">Loading chart...</div>
              </v-overlay>
            </v-card>
          </v-col>
        </v-row>
      </v-tab-item>

      <v-tab-item key="employees" class="pa-4">
        <v-card class="elevation-2 pa-4 mb-6 employee-card">
          <h2 class="text-h5 font-weight-bold primary--text mb-2">👥 Sales Employees</h2>
          <p class="subtitle-1 grey--text text--darken-1">Manage and track your sales team's performance.</p>
        </v-card>

        <v-card class="elevation-5 data-table-card">
          <v-toolbar flat class="table-toolbar">
            <v-toolbar-title class="font-weight-bold text-subtitle-1">Employee List</v-toolbar-title>
            <v-spacer></v-spacer>
            <v-text-field
              v-model="employeeSearch"
              append-icon="mdi-magnify"
              label="Search Employees"
              single-line
              hide-details
              clearable
              dense
              class="search-field"
            ></v-text-field>
            <v-btn color="primary" v-if="sales_generate_reports" dark class="ml-4 generate-report-btn" @click="showModal = true">
              <v-icon left>mdi-file-chart</v-icon> Generate Report
            </v-btn>
          </v-toolbar>

          <v-data-table
            :headers="employeeHeaders"
            :items="salesEmployees"
            class="employee-table"
            dense
            :page.sync="currentPage"
            :items-per-page="pagination.per_page"
            hide-default-footer
          >
            <template v-slot:item.performance_assessment="{ item }">
              <v-chip
                :color="getAssessmentColor(item.performance_assessment)"
                dark
                small
                label
                class="assessment-chip"
              >
                {{ item.performance_assessment }}
              </v-chip>
            </template>
            <template v-slot:item.employee_id="{ item }">
              <v-chip
                @click="getreport(item.employee_id)"
                color="blue-grey lighten-5"
                text-color="blue-grey darken-4"
                small
                label
                class="employee-id-chip"
              >
                {{ item.employee_id }}
              </v-chip>
            </template>
          </v-data-table>
          <v-pagination
            v-if="pagination.last_page > 1"
            v-model="currentPage"
            :length="pagination.last_page"
            @input="fetchSales"
            color="primary"
            class="mt-4 pb-2"
          />
        </v-card>
      </v-tab-item>
      <v-tab-item key="approved" class="pa-4">
        <v-card class="elevation-2 pa-4 mb-6 approved-bonus-card">
          <h2 class="text-h5 font-weight-bold primary--text mb-2">
            ✅ Sales Approved Product Bonus
          </h2>
          <p class="subtitle-1 grey--text text--darken-1">
            View and download the latest approved product bonus details.
          </p>
        </v-card>

        <v-card class="elevation-5 data-table-card">
          <v-toolbar flat class="table-toolbar">
            <v-toolbar-title class="font-weight-bold text-subtitle-1"
              >Approved Bonus List</v-toolbar-title
            >
            <v-spacer></v-spacer>
            <v-text-field
              v-model="approvedBonusSearch"
              append-icon="mdi-magnify"
              label="Search Approved Bonuses"
              single-line
              hide-details
              clearable
              dense
              class="search-field"
            ></v-text-field>
            <v-btn
              color="primary"
              dark
              class="ml-4 download-pdf-btn"
              @click="downloadApprovedBonusPdf"
              :disabled="!approvedBonusPdfUrl"
            >
              <v-icon left>mdi-file-pdf-box</v-icon> Download PDF
            </v-btn>
          </v-toolbar>

          <v-data-table
            :headers="approvedBonusHeaders"
            :items="approvedBonuses"
            class="approved-bonus-table"
            dense
            hide-default-footer
          >
            <template v-slot:item.bonus_amount="{ item }">
              <v-chip color="green lighten-4" text-color="green darken-4" small label>
                ₱ {{ parseFloat(item.bonus_amount).toLocaleString() }}
              </v-chip>
            </template>
             <template v-slot:item.as_of_date="{ item }">
              <v-chip color="blue-grey lighten-5" text-color="blue-grey darken-4" small label>
                {{ item.as_of_date ? item.as_of_date : 'N/A' }}
              </v-chip>
            </template>
            <template v-slot:item.Download="{ item }">
              <v-chip color="blue-grey lighten-5" text-color="blue-grey darken-4" small label>
                 <a :href="item.url">Download</a>
              </v-chip>
            </template>
          </v-data-table>
        </v-card>
      </v-tab-item>
      <v-tab-item key="sales" class="pa-4">
        <v-card class="elevation-2 pa-4 mb-6 product-card">
          <h2 class="text-h5 font-weight-bold primary--text mb-2">📦 Product Maintenance</h2>
          <p class="subtitle-1 grey--text text--darken-1">Manage product bonus configurations.</p>
        </v-card>

        <v-card class="elevation-5 data-table-card">
          <v-toolbar flat class="table-toolbar">
            <v-toolbar-title class="font-weight-bold text-subtitle-1">Product List</v-toolbar-title>
            <v-spacer></v-spacer>
            <v-text-field
              v-model="productSearch"
              append-icon="mdi-magnify"
              label="Search Products"
              single-line
              hide-details
              clearable
              dense
              class="search-field"
            ></v-text-field>
            <v-btn color="success" dark class="ml-4" @click="exportTemplates">
              <v-icon left>mdi-file-download</v-icon> Download Template
            </v-btn>
            <v-btn color="primary" dark class="ml-2" @click="dialogProductBonus = true">
              <v-icon left>mdi-upload</v-icon> Upload Bonus
            </v-btn>
          </v-toolbar>

          <v-data-table
            :headers="productHeaders"
            :items="products"
            class="product-table"
            dense
            hide-default-footer
          >
            <template v-slot:item.product_bonus="{ item }">
              <v-chip color="green lighten-4" text-color="green darken-4" small label>
                ₱ {{ parseFloat(item.product_bonus).toLocaleString() }}
              </v-chip>
            </template>
          </v-data-table>
        </v-card>
      </v-tab-item>


       



      
    </v-tabs-items>

    <v-dialog v-model="showModal" max-width="1200px">
      <v-card class="report-modal-card">
        <v-toolbar color="primary" dark flat class="dialog-toolbar">
          <v-icon left>mdi-file-document</v-icon>
          <v-toolbar-title class="font-weight-bold">Sales Report Generator</v-toolbar-title>
          <v-spacer></v-spacer>
          <v-btn
            color="success"
            dark
            class="export-excel-btn"
            v-if="reportData.length !== 0"
            @click="exportToExcel"
          >
            <v-icon left>mdi-file-excel</v-icon>Export to Excel
          </v-btn>
          <v-btn icon dark @click="showModal = false">
            <v-icon>mdi-close</v-icon>
          </v-btn>
        </v-toolbar>

        <v-card-text class="py-5 px-6">
          <v-row dense>
            <v-col cols="12" sm="6">
              <v-select
                v-model="reportType"
                :items="reportTypes"
                label="Type of Report"
                outlined
                dense
                hide-details
                color="primary"
              ></v-select>
            </v-col>
            <v-col cols="12" sm="6">
              <v-select
                v-model="selectedBranch"
                :items="branches"
                label="Select Branch"
                outlined
                dense
                hide-details
                color="primary"
                @change="check()"
              ></v-select>
            </v-col>
            <v-col cols="12" sm="6">
              <v-menu
                ref="fromMenu"
                v-model="fromMenu"
                :close-on-content-click="false"
                transition="scale-transition"
                offset-y
                min-width="auto"
              >
                <template v-slot:activator="{ on, attrs }">
                  <v-text-field
                    v-model="form.startDate"
                    label="Start Date"
                    prepend-inner-icon="mdi-calendar"
                    readonly
                    v-bind="attrs"
                    v-on="on"
                    outlined
                    dense
                    hide-details
                    @change="check()"
                    color="primary"
                  />
                </template>
                <v-date-picker v-model="form.startDate" @input="fromMenu = false" color="primary"></v-date-picker>
              </v-menu>
            </v-col>
            <v-col cols="12" sm="6">
              <v-menu
                ref="toMenu"
                v-model="toMenu"
                :close-on-content-click="false"
                transition="scale-transition"
                offset-y
                min-width="auto"
              >
                <template v-slot:activator="{ on, attrs }">
                  <v-text-field
                    v-model="form.endDate"
                    label="End Date"
                    prepend-inner-icon="mdi-calendar"
                    readonly
                    v-bind="attrs"
                    v-on="on"
                    outlined
                    dense
                    hide-details
                    color="primary"
                    @change="check()"
                  />
                </template>
                <v-date-picker v-model="form.endDate" @input="toMenu = false" color="primary"></v-date-picker>
              </v-menu>
            </v-col>
          </v-row>

          <v-btn
            color="primary"
            class="mt-6 generate-btn"
            block
            @click="generateReport(0)"
            :loading="loading"
            large
          >
            <v-icon left>mdi-play-circle</v-icon> Generate Report
          </v-btn>
          <v-btn
          v-if="sales_approved_bonus"
            color="success"
            class="mt-6 generate-btn"
            block
            @click="generateReport(1)"
            :loading="loading"
            large
            :disabled="approved"
          >
            <v-icon left>mdi-play-circle</v-icon> APPROVED
          </v-btn>

          <v-divider class="my-6 divider"></v-divider>

          <div v-if="download_link" class="mb-4">
            <strong class="primary--text">Download Link:</strong>
            <a :href="download_link" target="_blank" class="download-link ml-2">
              <v-icon small color="blue">mdi-link</v-icon> {{ filename }}
            </a>
          </div>

          <v-data-table :headers="headers" :items="reportData" class="report-data-table elevation-1" dense>
            <template v-slot:no-data>
              <v-alert type="info" text class="my-4">No report data yet. Generate a report above.</v-alert>
            </template>
          </v-data-table>
        </v-card-text>
      </v-card>
    </v-dialog>

    <v-dialog v-model="dialogProductBonus" max-width="500px">
      <v-card class="upload-modal-card">
        <v-toolbar color="primary" dark flat>
          <v-icon left>mdi-upload</v-icon>
          <v-toolbar-title class="font-weight-bold">Upload Product Bonus Template</v-toolbar-title>
          <v-spacer></v-spacer>
          <v-btn icon dark @click="dialogProductBonus = false">
            <v-icon>mdi-close</v-icon>
          </v-btn>
        </v-toolbar>

        <v-card-text class="py-5 px-6">
          <v-file-input
            v-model="file"
            label="Select Excel File"
            accept=".xlsx, .xls"
            outlined
            dense
            prepend-icon="mdi-file-excel"
            color="primary"
            class="file-input"
          ></v-file-input>

          <div v-if="file" class="mt-2 text-center">
            <v-chip color="green" text-color="white" small label>
              <v-icon left small>mdi-check-circle</v-icon> {{ file.name }}
            </v-chip>
          </div>
        </v-card-text>

        <v-card-actions class="px-6 pb-4">
          <v-spacer></v-spacer>
          <v-btn color="red" text @click="dialogProductBonus = false">Cancel</v-btn>
          <v-btn color="primary" :disabled="!file" @click="uploadFile" :loading="loading">
            <v-icon left>mdi-cloud-upload</v-icon>Upload
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <v-dialog v-model="salesdialog" max-width="1000px" persistent>
      <v-card class="sales-details-modal-card">
        <v-toolbar dark color="indigo" flat>
          <v-icon left>mdi-clipboard-list-outline</v-icon>
          <v-toolbar-title class="font-weight-bold">Product Sales List</v-toolbar-title>
          <v-spacer></v-spacer>
          <v-btn icon dark @click="salesdialog = false">
            <v-icon>mdi-close</v-icon>
          </v-btn>
        </v-toolbar>

        <v-card-text class="py-5 px-6">
          <v-text-field
            v-model="search"
            label="Search Sales"
            class="mb-4 search-field"
            append-icon="mdi-magnify"
            single-line
            hide-details
            outlined
            dense
            clearable
            color="primary"
          ></v-text-field>

          <v-data-table
            :headers="Salesheaders"
            :items="sales"
            class="elevation-1 sales-details-table"
            dense
          >
            <template v-slot:item.Amt="{ item }">
              <span class="font-weight-bold text--primary">₱ {{ parseFloat(item.Amt).toLocaleString() }}</span>
            </template>
            <template v-slot:no-data>
              <v-alert type="info" text class="my-4">No sales data available for this employee.</v-alert>
            </template>
          </v-data-table>
        </v-card-text>

        <v-card-actions class="px-6 pb-4">
          <v-spacer></v-spacer>
          <v-btn color="primary" @click="salesdialog = false">Close</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-container>
</template>
  
  <script>
  import * as XLSX from 'xlsx';
  import { saveAs } from 'file-saver';
  import axios from 'axios';
import { computed } from 'vue';
 
   
 
  export default {
    data() {
      return {
        approved: false,
        approvedBonusSearch: '',
        approvedBonuses: [],  
        approvedBonusHeaders: [
        { text: 'Branch', value: 'branch' },
        { text: 'DocName', value: 'docname' },
        { text: 'As Of Date', value: 'as_of_date' },
        { text: 'Download', value: 'Download' }
        ],
        roles: [],
        imageUrl:
        "https://cdn-sfx-linux-file-distribution-network-zone9-node8.addessa.com/download/graph?q=sales_performance_chart-ALL-2025-06-26.png&session=None",
        filename: '',
        download_link: '',
        search: '',
        salesdialog: false,
        sales: [  {
          DocDate: '2025-02-14',
          ItemCode: 'I-003666',
          ItemName: 'MS2535GIB',
          Brand: 'LG',
          Supplier: 'LG ELECTRONICS PHILS. INC.',
          Amt: '6500.00',
        }],
        Salesheaders: [
        { text: 'Date', value: 'DocDate' },
        { text: 'Item Code', value: 'ItemCode' },
        { text: 'Item Name', value: 'ItemName' },
        { text: 'Brand', value: 'Brand' },
        { text: 'Supplier', value: 'Supplier' },
        { text: 'Amount', value: 'Amt', align: 'right' },
        ],
        file: null,
        dialogProductBonus: false,
        loading: false,
        salesData: [],
        pagination: {},
        currentPage: 1,
        tab: 0,
        productSearch: '',
        employeeSearch: '',
        productHeaders: [
          { text: 'Brand', value: 'brand' },
          { text: 'Model', value: 'model' },
          { text: 'Product Bonus', value: 'product_bonus' },
        ],
        employeeHeaders: [
          { text: 'ID', value: 'employee_id' },
          { text: 'Name', value: 'employee' },
          { text: 'Date Hired', value: 'datehired' },
          { text: 'Position', value: 'position' },
          { text: 'Brand', value: 'brand' },
          // { text: 'Allied Sales Quota', value: 'allied_sales_qouta' },
          // { text: 'Sales Performance', value: 'sales_performance' },
          // { text: 'Product Bonus Total', value: 'product_bonus_total' },
          // { text: 'Action', value: 'action' },
        ],
        products: [
      
        ],
        salesEmployees: [ 
        ],
        reportType: '',
        selectedBranch: '',
        reportTypes: ['Quarterly Evaluation', 'Product Bonus'],
        branches: [ ],

      showModal: false,
      fromMenu: false,
      toMenu: false,
      form: {
        startDate: null,
        endDate: null,
      },
      reportData: [],
      headers: [
        { text: "Branch", value: "branch" },
        { text: "Employee", value: "employee" },
        { text: "Date Hired", value: "datehired" },
        { text: "Brand", value: "brand" },
        { text: "Quota", value: "qouta" },
        { text: "Sales Quota", value: "sale_qouta" },
        { text: "Sales Performance", value: "sales_performance" },
        { text: "Performance Assessment", value: "peformance_assessment" },
        { text: "Recommendation", value: "recommendation" },
      ],
      };
    },
    computed:{
      sales_download_reports(){
        return this.roles.includes('Sales Download Reports');
      },
      sales_generate_reports(){
        return this.roles.includes('Sales Generate Reports');
      },
      sales_user(){
        return this.roles.includes('Sales User');
      },
      sales_admin(){
        return this.roles.includes('Sales Admin');
      },
      sales_upload_product(){
        return this.roles.includes('Sales Upload Product');
      },
      sales_approved_bonus(){
        return this.roles.includes('Sales Approved Bonus');
      },
    },
    methods: {
     
      check(){
        axios
        .get('http://10.10.10.40:8083/api/sales/smi/check?start_date='+this.form.startDate+'&end_date='+this.form.endDate+'&q=1&branch='+this.selectedBranch)
        .then((res) => {
           if(res.data > 0){
              this.loading = false
              this.approved = true
              return this.approved
           }else{
            this.approved = false
           }
          
        })
        .catch((error) => {
          console.error('❌ Error fetching data:', error);
        });
      },
      getreport(item){
        axios
        .get('http://10.10.10.40:8083/api/sales/smi/sale/list?data='+item)
        .then((res) => {
          this.salesdialog = true;
          this.sales = res.data
        })
        .catch((error) => {
          console.error('❌ Error fetching data:', error);
        });
      },
      getAssessmentColor(assessment) {
        switch (assessment) {
          case 'Excellent':
            return 'green';
          case 'Good':
            return 'blue';
          case 'Average':
            return 'orange';
          case 'Poor':
            return 'red';
          default:
            return 'grey';
        }
      },
      fetchSales(page = 1) {
         axios.get(`http://10.10.10.40:8083/api/sales/smi/index?page=${page}`).then((res) => {
          this.salesEmployees = res.data.data;
          this.pagination = {
            total: res.data.total,
            per_page: res.data.per_page,
            current_page: res.data.current_page,
            last_page: res.data.last_page,
            next_page_url: res.data.next_page_url,
            prev_page_url: res.data.prev_page_url,
          };
        });
    },
    async generateReport(i) {
      var identify;
      if (!this.form.startDate || !this.form.endDate) {
        alert("Please select a date range.");
        return;
      }

      if(this.reportType == 'Quarterly Evaluation'){
        identify = 0
           this.headers = [
            { text: "Branch", value: "branch" },
            { text: "Employee", value: "employee" },
            { text: "Date Hired", value: "datehired" },
            { text: "Brand", value: "brand" },
            { text: "Quota", value: "qouta" },
            { text: "Sales Quota", value: "sale_qouta" },
            { text: "Sales Performance", value: "sales_performance" },
            { text: "Performance Assessment", value: "peformance_assessment" },
            { text: "Recommendation", value: "recommendation" },
           ]

      }else{
        identify = 1
        this.headers = [
            { text: "Branch", value: "branch" },
            { text: "Employee", value: "employee" },
            { text: "Date Hired", value: "datehired" },
            { text: "Brand", value: "brand" },
            { text: "Quota", value: "qouta" },
            { text: "Sales Quota", value: "sale_qouta" },
            { text: "Sales Performance", value: "sales_performance" },
            { text: "Product Bonus Total", value: "product_bonus_total" },
            { text: "Received by/Date:", value: "received" },
           ]
      } 
      var ind ;
      if(i == 0){
        ind = 'generator';
      }else{ 
        ind = 'approved';
        if(this.check()){
          return;
        }
      }

      try {
        this.loading = true
        const res = await axios.get("http://10.10.10.40:8083/api/sales/smi/"+ind, {
          params: {
            start_date: this.form.startDate,
            end_date: this.form.endDate,
            q: identify,
            branch: this.selectedBranch
          },
        });
        if(i == 1){
          this.loading = false
          alert('Approved')
        }else{
          this.reportData = res.data.reports;
        this.download_link = res.data.download_link
        this.filename = res.data.filename
        this.loading = false
        }
         
      } catch (err) {
        console.error("Error fetching report:", err);
      }
    },
    exportToExcel() {
      var data 
      if(this.reportType == 'Quarterly Evaluation'){
           data = this.reportData.map(row => {
              return {
                  Branch: row.branch,
                  Employee: row.employee,
                  DateHired: row.datehired,
                  Brand: row.brand,
                  Quota: row.qouta,
                  SalesQuota: row.sale_qouta,
                  SalesPerformance: row.sales_performance,
                  PerformanceAssessment: row.peformance_assessment,
                  Recommendation: row.recommendation,
            };
          });
        }else{
          data = this.reportData.map(row => {
              return {
                  Branch: row.branch,
                  Employee: row.employee,
                  DateHired: row.datehired,
                  Brand: row.brand,
                  Quota: row.qouta,
                  SalesQuota: row.sale_qouta,
                  SalesPerformance: row.sales_performance,
                  ProductBonusTotal: row.product_bonus_total,
                  ReceivedbyDate: ''
            };
          });

        }
      const worksheet = XLSX.utils.json_to_sheet(data);
      const workbook = XLSX.utils.book_new();
      XLSX.utils.book_append_sheet(workbook, worksheet, "Report");
      const excelBuffer = XLSX.write(workbook, {
        bookType: "xlsx",
        type: "array"
      });
      const blob = new Blob([excelBuffer], { type: "application/octet-stream" });
      saveAs(blob, "sales_report.xlsx");
    },
    exportTemplates() {
      var data 
        data = this.products.map(row => {
            return {
                Brand: row.brand,
                Model: row.model,
                Product_Bonus: row.product_bonus,
            };
        });
     
      const worksheet = XLSX.utils.json_to_sheet(data);
      const workbook = XLSX.utils.book_new();
      XLSX.utils.book_append_sheet(workbook, worksheet, "Report");
      const excelBuffer = XLSX.write(workbook, {
        bookType: "xlsx",
        type: "array"
      });

      const blob = new Blob([excelBuffer], { type: "application/octet-stream" });
      saveAs(blob, "product_bonus_templates.xlsx");
    },
    uploadFile() {
      if (!this.file) return;

      const formData = new FormData();
      formData.append("file", this.file);

       axios.post("http://10.10.10.40:8083/api/sales/smi/upload-product-bonus", formData, {
        headers: { "Content-Type": "multipart/form-data" }
      })
      .then(() => {
        this.$emit("uploaded");
        this.dialogProductBonus = false;
        this.file = null;
       
      })
      .catch((err) => {
        console.error(err);
        this.$toast.error("Upload failed");
      });
    }
    },
    mounted() {
 
      axios.get('http://10.10.10.40:8083/api/sales/smi/getbonus').then((res)=>{
          this.approvedBonuses = res.data
      })
      axios.get('http://10.10.10.40:8083/api/sales/smi/graph').then((res)=>{
          this.imageUrl = res.data
      })
      axios.post('http://10.10.10.40:8083/api/auth/permissions')
      .then((res) => {
        this.roles = res.data
        console.log(this.roles)
      });

      axios
        .get('http://10.10.10.40:8083/api/sales/smi/index')
        .then((res) => {
     
          this.salesEmployees = res.data.data
          this.pagination = {
            total: res.data.total,
            per_page: res.data.per_page,
            current_page: res.data.current_page,
            last_page: res.data.last_page,
            next_page_url: res.data.next_page_url,
            prev_page_url: res.data.prev_page_url,
          };
          
        })
        .catch((error) => {
          console.error('❌ Error fetching data:', error);
        });
         
        axios
        .get('http://10.10.10.40:8083/api/sales/smi/branch')
        .then((res) => {
            
            this.branches = []
        
           res.data.forEach((i,s) => {
                 this.branches.push(i.Branch) 
           });
           
        })
        .catch((error) => {
          console.error('❌ Error fetching data:', error);
        });

        axios
        .get('http://10.10.10.40:8083/api/sales/smi/items')
        .then((res) => {
       
          this.products = res.data
 
           
        })
        .catch((error) => {
          console.error('❌ Error fetching data:', error);
        });
    }


    
  };
  </script>
  <style scoped>
  .v-toolbar-title {
    font-weight: bold;
    font-size: 20px;
  }
  </style>
  ---

  ### **Comprehensive Styling for Your Components**
  
  ```css
  <style scoped>
  /* Main Container */
  .v-container.fluid.pa-0 {
    background-color: #f5f7fa; /* Light background for the overall content area */
  }
  
  /* --- Tabs Styling --- */
  .sales-edge-tabs {
    background: linear-gradient(to right, #007bff, #0056b3) !important; /* Matches app bar gradient */
    box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
    border-radius: 8px 8px 0 0; /* Rounded top corners */
    margin-bottom: 20px; /* Space below tabs */
  }
  
  .sales-edge-tabs .v-tab {
    color: rgba(255, 255, 255, 0.7) !important; /* Muted white for inactive tabs */
    font-weight: 500;
    text-transform: capitalize; /* More natural capitalization */
    font-size: 1rem;
    letter-spacing: 0.5px;
    transition: all 0.3s ease;
  }
  
  .sales-edge-tabs .v-tab:hover {
    background-color: rgba(255, 255, 255, 0.1);
    color: #ffffff !important; /* Brighter white on hover */
  }
  
  .sales-edge-tabs .v-tab--active {
    color: #ffffff !important; /* Pure white for active tab */
    background-color: rgba(255, 255, 255, 0.15); /* Slightly darker active background */
    font-weight: bold;
  }
  
  .sales-edge-tabs .v-tabs-slider {
    background-color: #ffeb3b !important; /* Vibrant yellow slider for active tab */
    height: 4px;
  }
  
  /* --- Tab Item Card Base Styles --- */
  .dashboard-card,
  .employee-card,
  .product-card {
    border-radius: 12px;
    background: linear-gradient(to right, #e3f2fd, #bbdefb); /* Light blue gradient for header cards */
    color: #1a237e; /* Dark blue text */
    text-align: center;
    padding: 25px !important;
    box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1) !important;
  }
  
  .dashboard-card h2,
  .employee-card h2,
  .product-card h2 {
    font-size: 2rem;
    margin-bottom: 10px;
  }
  
  .dashboard-card p,
  .employee-card p,
  .product-card p {
    font-size: 1.1rem;
  }
  
  /* --- Dashboard Specific Styles --- */
  .chart-card {
    border-radius: 12px;
    overflow: hidden;
    box-shadow: 0 8px 25px rgba(0, 0, 0, 0.15) !important;
  }
  
  .chart-image {
    background-color: #ffffff; /* Ensure white background if image has transparency */
    display: block; /* Remove extra space below image */
  }
  
  /* --- Data Table Card Styles (Employees & Products) --- */
  .data-table-card {
    border-radius: 12px;
    overflow: hidden; /* Ensures rounded corners on inner elements */
    box-shadow: 0 5px 20px rgba(0, 0, 0, 0.1);
  }
  
  .table-toolbar {
    background-color: #f8f9fa !important; /* Light background for table toolbars */
    border-bottom: 1px solid #e0e0e0;
    padding: 10px 20px;
  }
  
  .table-toolbar .v-toolbar__title {
    color: #333;
  }
  
  .search-field {
    max-width: 300px; /* Limit search field width */
  }
  
  .generate-report-btn,
  .export-template-btn,
  .upload-bonus-btn {
    letter-spacing: 0.5px;
    font-weight: 600;
  }
  
  /* Table Specifics */
  .employee-table,
  .product-table {
    background-color: #ffffff;
  }
  
  .v-data-table >>> th {
    background-color: #f0f4f8 !important; /* Light blue header background */
    color: #3f51b5 !important; /* Darker blue header text */
    font-weight: bold;
    font-size: 0.95rem;
    text-transform: uppercase;
    letter-spacing: 0.5px;
  }
  
  .v-data-table >>> td {
    font-size: 0.9rem;
    color: #424242;
  }
  
  .v-data-table >>> tbody tr:nth-of-type(odd) {
    background-color: #f9f9f9;
  }
  
  .v-data-table >>> tbody tr:hover {
    background-color: #eef5fc !important; /* Light hover background */
    cursor: pointer;
  }
  
  .assessment-chip {
    font-weight: bold;
    min-width: 90px;
    justify-content: center;
  }
  
  .employee-id-chip {
    font-weight: bold;
    cursor: pointer;
    transition: background-color 0.2s;
  }
  
  .employee-id-chip:hover {
    background-color: #e0f2f7 !important; /* Lighter on hover */
  }
  
  /* Pagination */
  .v-pagination .v-pagination__item,
  .v-pagination .v-pagination__navigation {
    box-shadow: none !important;
    border: 1px solid #e0e0e0;
    margin: 0 4px;
    border-radius: 8px;
  }
  
  .v-pagination .v-pagination__item--active {
    background-color: #007bff !important;
    color: white !important;
  }
  
  /* --- Modal Dialogs General Styling --- */
  .report-modal-card,
  .upload-modal-card,
  .sales-details-modal-card {
    border-radius: 16px;
    overflow: hidden;
    box-shadow: 0 10px 40px rgba(0, 0, 0, 0.25);
  }
  
  .dialog-toolbar {
    background: linear-gradient(to right, #007bff, #0056b3) !important; /* Matches app bar */
    padding: 10px 20px;
    height: auto !important; /* Allow height to adjust */
    min-height: 56px; /* Standard toolbar min-height */
  }
  
  .dialog-toolbar .v-toolbar__title {
    font-size: 1.25rem;
    letter-spacing: 0.5px;
    color: white;
  }
  
  .export-excel-btn {
    font-weight: 600;
    letter-spacing: 0.5px;
  }
  
  .report-modal-card .v-card__text,
  .upload-modal-card .v-card__text,
  .sales-details-modal-card .v-card__text {
    background-color: #fdfdfd; /* Off-white background for content */
  }
  
  /* Report Generator Specifics */
  .generate-btn {
    font-weight: bold;
    letter-spacing: 1px;
    font-size: 1.1rem;
    padding: 25px 0 !important; /* Increase button height */
    border-radius: 8px;
    box-shadow: 0 4px 10px rgba(0, 123, 255, 0.3) !important;
    transition: all 0.3s ease;
  }
  
  .generate-btn:hover {
    transform: translateY(-2px);
    box-shadow: 0 6px 15px rgba(0, 123, 255, 0.4) !important;
  }
  
  .divider {
    border-color: rgba(0, 0, 0, 0.1) !important; /* Lighter divider */
  }
  
  .download-link {
    color: #007bff !important;
    font-weight: 500;
    text-decoration: none;
    transition: color 0.2s;
  }
  
  .download-link:hover {
    color: #0056b3 !important;
    text-decoration: underline;
  }
  
  .report-data-table {
    background-color: #ffffff;
    border-radius: 8px;
  }
  
  /* File Input Specifics */
  .file-input .v-input__control {
    border-radius: 8px;
  }
  
  /* Sales Details Modal Specifics */
  .sales-details-modal-card .search-field {
    max-width: 100%; /* Full width for search in dialog */
  }
  
  .sales-details-table {
    border-radius: 8px;
  }
  </style>