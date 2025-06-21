<template>
    <v-container>
      <!-- Tabs -->
      <v-tabs v-model="tab" background-color="primary" dark>
        <v-tab key="dashboard">Dashboard</v-tab>
        <v-tab key="employees">Employees</v-tab>
        <v-tab key="sales">Sales</v-tab>
      </v-tabs>
  
      <v-tabs-items v-model="tab">
        <!-- Dashboard -->
        <v-tab-item key="dashboard">
          <v-card flat class="pa-4">
            <h2>📊 Dashboard</h2>
            <p>Performance overview and analytics.</p>
          </v-card>
        </v-tab-item>
  
        <!-- Employees with Sales Employee Table -->
        <v-tab-item key="employees">
          <v-card flat class="pa-4">
            <h2>👥 Sales Employees</h2>
            
              <!-- Toolbar with Sync Button -->
                <v-toolbar flat color="white">
                   
                 
                    <v-toolbar-title>Employee List</v-toolbar-title>
                    <v-spacer></v-spacer>
                    <v-btn color="primary" style="margin-right: 10px;" dark @click="showModal = true">
                      📊 Generate Report
                    </v-btn>
          
                    
                    <v-btn color="primary" dark @click="syncEmployees">
                    <v-icon left>mdi-sync</v-icon>
                    Sync Query
                    </v-btn>
                     
                </v-toolbar>

                 
            <v-data-table
              :headers="employeeHeaders"
              :items="salesEmployees"
              class="elevation-1"
              dense
              hide-default-footer
            >
              <template v-slot:top>
                <v-toolbar flat color="white">
               
                  <v-spacer></v-spacer>
                  <v-text-field
                    v-model="employeeSearch"
                    append-icon="mdi-magnify"
                    label="Search"
                    single-line
                    hide-details
                  ></v-text-field>
                </v-toolbar>
              </template>
  
              <template v-slot:item.performance_assessment="{ item }">
                <v-chip
                  :color="getAssessmentColor(item.performance_assessment)"
                  text-color="white"
                  small
                >
                  {{ item.performance_assessment }}
                </v-chip>
              </template>
              <template v-slot:item.employee_id="{ item }">
                <v-chip @click="getreport()" color="green lighten-4" text-color="green darken-4" small>
                  {{ item.employee_id }}
                </v-chip>
              </template>
            </v-data-table>
            <v-pagination
              v-if="pagination.last_page > 1"
              v-model="currentPage"
              :length="pagination.last_page"
              @input="fetchSales"
            />
          </v-card>
        </v-tab-item>
  
        <!-- Sales with Product Table -->
        <v-tab-item key="sales">
          <v-card flat class="pa-4">
            <h2>Product Maitenance</h2>
            <v-spacer></v-spacer>
              <v-btn color="green darken-1" @click="exportTemplates" style="margin: 10px;">
                Download Templates
              </v-btn>
                 <!-- Upload Trigger Button -->
              <v-btn color="primary" @click="dialogProductBonus = true">
                Upload Product Bonus Template
              </v-btn>
            <v-data-table
              :headers="productHeaders"
              :items="products"
              class="elevation-1"
              dense
              hide-default-footer
            >
              <template v-slot:top>
                <v-toolbar flat color="white">
                  <v-toolbar-title>Product List</v-toolbar-title>
                  <v-spacer></v-spacer>
                  <v-text-field
                    v-model="productSearch"
                    append-icon="mdi-magnify"
                    label="Search"
                    single-line
                    hide-details
                  ></v-text-field>
                </v-toolbar>
              </template>
  
              <template v-slot:item.product_bonus="{ item }">
                <v-chip color="green lighten-4" text-color="green darken-4" small>
                  {{ item.product_bonus }}
                </v-chip>
              </template>
            </v-data-table>
          </v-card>
        </v-tab-item>
      </v-tabs-items>
         <!-- Modal Dialog -->
        <v-dialog v-model="showModal" max-width="1200px">
          <v-card>

            
            <v-card-title class="headline grey lighten-2">
              <v-icon left>mdi-file-document</v-icon>
                Sales Report Generator
                <v-spacer ></v-spacer>
                <v-btn color="green darken-1" v-if="reportData.length !==0" @click="exportToExcel">
                Export to Excel
              </v-btn>

            </v-card-title>
            
            <v-card-text>
              <v-row dense>

              
                <v-col cols="12" sm="6" >
                  <v-select
                    v-model="reportType"
                    :items="reportTypes"
                    label="Type of Report"
                    outlined
                    dense
                    style="margin-top: 10px;"
                  ></v-select>
                </v-col>

                <v-col cols="12" sm="6">
                  <v-select
                    v-model="selectedBranch"
                    :items="branches"
                    label="Select Branch"
                    outlined
                    dense
                      style="margin-top: 10px;"
                  ></v-select>
                </v-col>
          
                <v-col cols="12" sm="6">
                  <v-menu ref="fromMenu" v-model="fromMenu" :close-on-content-click="false" transition="scale-transition" offset-y>
                    <template v-slot:activator="{ on, attrs }">
                      <v-text-field v-model="form.startDate" label="Start Date" prepend-icon="mdi-calendar" readonly v-bind="attrs" v-on="on" />
                    </template>
                    <v-date-picker v-model="form.startDate" @input="fromMenu = false"></v-date-picker>
                  </v-menu>
                </v-col>

                <v-col cols="12" sm="6">
                  <v-menu ref="toMenu" v-model="toMenu" :close-on-content-click="false" transition="scale-transition" offset-y>
                    <template v-slot:activator="{ on, attrs }">
                      <v-text-field v-model="form.endDate" label="End Date" prepend-icon="mdi-calendar" readonly v-bind="attrs" v-on="on" />
                    </template>
                    <v-date-picker v-model="form.endDate" @input="toMenu = false"></v-date-picker>
                  </v-menu>
                </v-col>
              </v-row>

              <v-btn color="success" class="mt-3" block @click="generateReport" :loading="loading">
                <v-icon left>mdi-play</v-icon> Generate Report
              </v-btn>
              
              <v-divider class="my-4"></v-divider>
          
              <!-- Data Table -->
              <v-data-table :headers="headers" :items="reportData" dense outlined item-value="employee">
                <template v-slot:no-data>
                  <v-alert type="info" text> No report data yet. </v-alert>
                </template>
              </v-data-table>
            </v-card-text>

            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn text color="red" @click="showModal = false">
                Close
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
            <!-- Modal Dialog -->
     <v-dialog v-model="dialogProductBonus" max-width="500px">
        <v-card>
          <v-card-title>
            <span class="headline">Upload Product Bonus Template</span>
          </v-card-title>

          <v-card-text>
            <v-file-input
              v-model="file"
              label="Select Excel File"
              accept=".xlsx, .xls"
              outlined
              dense
              prepend-icon="mdi-upload"
            ></v-file-input>

            <div v-if="file" class="mt-2">
              <v-chip color="green" text-color="white" small>
                {{ file.name }}
              </v-chip>
            </div>
          </v-card-text>

          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="red" text @click="dialogProductBonus = false">Cancel</v-btn>
            <v-btn color="primary" :disabled="!file" @click="uploadFile">Upload</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>`
    </v-container>
  </template>
  
  <script>
  import * as XLSX from 'xlsx';
  import { saveAs } from 'file-saver';
  import axios from 'axios';
  export default {
    data() {
      return {
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
    methods: {
      getreport(){
         alert()
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
    async generateReport() {
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

      try {
        this.loading = true
        const res = await axios.get("http://10.10.10.40:8083/api/sales/smi/generator", {
          params: {
            start_date: this.form.startDate,
            end_date: this.form.endDate,
            q: identify,
            branch: this.selectedBranch
          },
        });
        
        this.reportData = res.data.reports;
        this.loading = false
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
      axios
        .get('http://10.10.10.40:8083/api/sales/smi/index')
        .then((res) => {
          console.log('✅ Response:', res.data);
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
            this.branches.push('ALL') 
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
  