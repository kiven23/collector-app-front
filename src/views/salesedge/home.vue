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
                  <v-toolbar-title>Employee List</v-toolbar-title>
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
            </v-data-table>
          </v-card>
        </v-tab-item>
  
        <!-- Sales with Product Table -->
        <v-tab-item key="sales">
          <v-card flat class="pa-4">
            <h2>💼 Sales</h2>
  
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
    </v-container>
  </template>
  
  <script>
  export default {
    data() {
      return {
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
          { text: 'Allied Sales Quota', value: 'allied_sales_qouta' },
          { text: 'Sales Performance', value: 'sales_performance' },
          { text: 'Product Bonus Total', value: 'product_bonus_total' },
          { text: 'Assessment', value: 'performance_assessment' },
        ],
        products: [
          { brand: 'Samsung', model: 'Galaxy A50', product_bonus: 'Free Case' },
          { brand: 'Apple', model: 'iPhone 14', product_bonus: 'Gift Card' },
        ],
        salesEmployees: [
          {
            employee_id: 'EMP001',
            employee: 'Juan Dela Cruz',
            datehired: '2023-05-01',
            position: 'Sales Executive',
            brand: 'Samsung',
            allied_sales_qouta: '100,000',
            sales_performance: '95,000',
            product_bonus_total: '5,000',
            performance_assessment: 'Good',
          },
          {
            employee_id: 'EMP002',
            employee: 'Maria Santos',
            datehired: '2022-11-15',
            position: 'Sales Associate',
            brand: 'Apple',
            allied_sales_qouta: '150,000',
            sales_performance: '160,000',
            product_bonus_total: '10,000',
            performance_assessment: 'Excellent',
          },
        ],
      };
    },
    methods: {
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
    },
  };
  </script>
  