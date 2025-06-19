<template>
    <v-app>
      <v-container class="pa-4" fluid>
        <v-card class="elevation-4">
          <v-card-title class="headline">Installment Application</v-card-title>
          <v-card-text>
            <v-stepper v-model="step">
              <v-stepper-header>
                <v-stepper-step :complete="step > 1" step="1">Applicant Info</v-stepper-step>
                <v-divider></v-divider>
                <v-stepper-step :complete="step > 2" step="2">Upload Documents</v-stepper-step>
                <v-divider></v-divider>
                <v-stepper-step step="3">Review & Submit</v-stepper-step>
              </v-stepper-header>
  
              <v-stepper-items>
                <v-stepper-content step="1">
                  <v-form ref="form">
                    <v-text-field v-model="form.full_name" label="Full Name" outlined dense required></v-text-field>
                    <v-text-field v-model="form.date_of_birth" label="Date of Birth" type="date" outlined dense required></v-text-field>
                    <v-select v-model="form.civil_status" :items="['Single', 'Married']" label="Civil Status" outlined dense></v-select>
                    <v-text-field v-model="form.barangay" label="Search Brgy" outlined dense @input="getFullListAddress()"></v-text-field>
                    <v-autocomplete
                      v-model="selectedAddress"
                      :loading="loadingAddress"
                      :items="listAddress"
                      :search-input.sync="searchAddress"
                      cache-items
                      
                      hide-no-data
                      hide-details
                      label="Brgy, City, Province, Region"
                      dense
                      outlined 
                      style="margin-bottom: 20px;"
                    ></v-autocomplete>
                    <v-text-field
                      v-model="form.contact_number"
                      label="Contact Number"
                      outlined 
                      dense
                      :rules="[validatePhone]"
                      maxlength="11"
                      @keypress="onlyPhilippinesNo($event)"
                    ></v-text-field>

                    <v-text-field v-model="form.employer_name" label="Employer Name" outlined dense></v-text-field>
                    <v-select
                      v-model="form.position"
                      :items="positions"
                      label="Position"
                      outlined
                      dense
                    ></v-select>                   
                    <v-select
                      v-model="form.monthly_income"
                      :items="incomeRanges"
                      label="Monthly Income Range"
                      outlined
                      dense
                    ></v-select>

                    <v-text-field
                      v-model="form.loan_amount_requested"
                      label="₱ Loan Amount Requested"
                      type="number"
                      outlined
                      dense
                      @keypress="onlyNumber($event)"
                    ></v-text-field>
                    <v-select
                      v-model="form.loan_term"
                      :items="loanTerms"
                      label="Loan Term (months)"
                      outlined
                      dense
                    ></v-select>                        
                    <v-select
                      v-model="form.purpose"
                      :items="productOptions"
                      label="Select Item to Purchase"
                      outlined
                      dense
                    ></v-select>

                    <!-- <v-text-field v-model="form.tin" label="TIN (optional)" outlined dense></v-text-field>
                    <v-text-field v-model="form.sss" label="SSS / GSIS Number (optional)" outlined dense></v-text-field> -->
                  </v-form>
                  <v-btn color="primary" class="mt-3" @click="nextStep">Next</v-btn>
                </v-stepper-content>
  
                <v-stepper-content step="2">
                  <v-file-input v-model="form.selfie" label="Selfie with ID" outlined dense accept="image/*"></v-file-input>
                  <v-file-input v-model="form.valid_id" label="Valid ID" outlined dense accept="image/*"></v-file-input>
                  <v-file-input v-model="form.proof_of_billing" label="Proof of Billing" outlined dense accept="image/*"></v-file-input>
                  <v-file-input v-model="form.payslip" label="Payslip / Proof of Income" outlined dense accept="image/*"></v-file-input>
                
                <!-- <v-file-input v-model="form.employment_certificate" label="Employment Certificate / Company ID" outlined dense accept="image/*"></v-file-input> -->
                <!-- <v-file-input v-model="form.collateral" label="Collateral Docs (if any)" outlined dense accept="image/*"></v-file-input> -->
                 <v-btn class="mt-3" @click="step--">Back</v-btn>
                <v-btn color="primary" class="mt-3 ml-2" @click="nextStep">Next</v-btn>
                </v-stepper-content>
  
                <v-stepper-content step="3">
                  <p class="text-subtitle-1">Review your info before submitting.</p>
                  <v-btn class="mt-2" @click="step--">Back</v-btn>
                  <v-btn color="green" class="mt-2 ml-2" @click="submitApplication">Submit</v-btn>
                </v-stepper-content>
              </v-stepper-items>
            </v-stepper>
          </v-card-text>
        </v-card>
      </v-container>
    </v-app>
  </template>
  
  <script>
  //  v-model="selectedAddress"
  //                     :loading="loadingAddress"
  //                     :items="listAddress"
  //                     :search-input.sync="searchAddress"
  export default {
    data: () => ({
      loadingAddress: false,
        selectedAddress: null,
        listAddress: [],
        searchAddress: null,
        allAddress: [
             "Poblacion, Agoncillo, Batangas, Region IV-A (CALABARZON)",
             "Poblacion, Aguilar, Pangasinan, Region I (Ilocos Region)",
             "Poblacion, Ajuy, Iloilo, Region VI (Western Visayas)",
             "Poblacion, Alabel, Sarangani, Region XII (SOCCSKSARGEN)",
             "Poblacion, Albuera, Leyte, Region VIII (Eastern Visayas)",
             "Poblacion, Amlan, Negros Oriental, Region VII (Central Visayas)"
        ],
      step: 1,
      form: {
        barangay: '',
        full_name: '',
        date_of_birth: '',
        civil_status: '',
        address: '',
        contact_number: '',
        employer_name: '',
        position: '',
        monthly_income: null,
        loan_amount_requested: null,
        loan_term: null,
        purpose: '',
        tin: '',
        sss: '',
        credit_history_notes: '',
        valid_id: null,
        proof_of_billing: null,
        employment_certificate: null,
        payslip: null,
        loan_form: null,
        collateral: null,
        selfie: null
      },
      positions: [
          'Software Developer',
          'Web Developer',
          'System Administrator',
          'IT Support',
          'Data Analyst',
          'Project Manager',
          'Sales Representative',
          'Customer Service',
          'Accountant',
          'Marketing Specialist',
          'HR Officer',
          'Operations Manager',
          'Warehouse Staff',
          'Delivery Driver',
          'Technician',
          'Cashier',
          'Store Supervisor',
          'Inventory Clerk',
          'Logistics Coordinator',
          'Procurement Officer',
          'Product Designer',
          'UI/UX Designer',
          'QA Tester',
          'Mobile App Developer',
          'Network Engineer',
          'Security Officer',
          'Receptionist',
          'Office Staff',
          'Janitor',
          'Electrician',
          'Mechanic',
          'Forklift Operator',
          'Production Worker',
          'Maintenance Staff',
          'Finance Officer',
          'Legal Assistant',
          'Content Creator',
          'Social Media Manager',
          'Trainer',
          'Field Auditor',
          'Branch Manager',
          'Area Manager',
          'Business Development Officer',
          'Administrative Assistant'
        ],
        positionsWithPoints: {
          'Software Developer': 90,
          'Web Developer': 85,
          'System Administrator': 88,
          'IT Support': 80,
          'Data Analyst': 90,
          'Project Manager': 95,
          'Sales Representative': 75,
          'Customer Service': 70,
          'Accountant': 88,
          'Marketing Specialist': 82,
          'HR Officer': 80,
          'Operations Manager': 92,
          'Warehouse Staff': 65,
          'Delivery Driver': 60,
          'Technician': 72,
          'Cashier': 60,
          'Store Supervisor': 78,
          'Inventory Clerk': 70,
          'Logistics Coordinator': 75,
          'Procurement Officer': 85,
          'Product Designer': 80,
          'UI/UX Designer': 83,
          'QA Tester': 82,
          'Mobile App Developer': 90,
          'Network Engineer': 85,
          'Security Officer': 60,
          'Receptionist': 65,
          'Office Staff': 68,
          'Janitor': 55,
          'Electrician': 70,
          'Mechanic': 68,
          'Forklift Operator': 60,
          'Production Worker': 60,
          'Maintenance Staff': 65,
          'Finance Officer': 90,
          'Legal Assistant': 88,
          'Content Creator': 75,
          'Social Media Manager': 78,
          'Trainer': 80,
          'Field Auditor': 87,
          'Branch Manager': 94,
          'Area Manager': 95,
          'Business Development Officer': 90,
          'Administrative Assistant': 70
        },
        loanTerms: [6, 12, 18, 24, 36, 48, 60],
        productOptions: [
          'Refrigerator',
          'Washing Machine',
          'Air Conditioner',
          'Television',
          'Gas Range',
          'Water Dispenser',
          'Furniture - Sofa Set',
          'Furniture - Dining Set',
          'Furniture - Bed Frame',
          'Electric Fan',
          'Microwave Oven',
          'Rice Cooker',
          'Blender',
          'Laptop',
          'Mobile Phone'
          ],
          incomeRanges: [
            'Below ₱5,000',
            '₱5,000 - ₱10,000',
            '₱10,001 - ₱15,000',
            '₱15,001 - ₱20,000',
            '₱20,001 - ₱30,000',
            '₱30,001 - ₱50,000',
            '₱50,001 - ₱70,000',
            '₱70,001 - ₱100,000',
            'Above ₱100,000'
          ]

    }),
    mounted(){
      
    },
    computed: {
   
    positionPoints() {
      return this.positionsWithPoints[this.form.position] || 0;
      }
    }
    ,
    watch: {
      searchAddress (val) {
        val && val !== this.selectedAddress && this.querySelections(val)
      },
    },
    methods: {
      
       querySelections (v) {
          this.loadingAddress = true
          // Simulated ajax query
          setTimeout(() => {
            this.listAddress = this.allAddress.filter(e => {
              return (e || '').toLowerCase().indexOf((v || '').toLowerCase()) > -1
            })
            this.loadingAddress = false
          }, 500)
        },
  
      getFullListAddress(){
          
          fetch('http://localhost:3000/find-address?q=' + this.form.barangay)
          .then(res => res.json()) // convert to JSON
          .then(data => {
            // var data2 = []
            // console.log(data)
            this.allAddress = []
            data.forEach(element => {
         
               this.allAddress.push(element.name)
            });
            // do something with data, like set to variable
            // this.addressResults = data (example)
          })
          .catch(err => {
            console.error('Error fetching address:', err)
          })
           
           
      
       },
      onlyPhilippinesNo(event) {
        const key = event.key;
        if (!/[0-9]/.test(key)) {
          event.preventDefault();
        }
      },
      validatePhone(value) {
        if (!value) return 'Contact number is required';
        if (!/^09\d{9}$/.test(value)) return 'Invalid format. Use 09XXXXXXXXX';
        return true;
      },
      onlyNumber(event) {
        const keyCode = event.keyCode ? event.keyCode : event.which;
        // Allow only numbers (0–9)
        if (keyCode < 48 || keyCode > 57) {
          event.preventDefault();
        }
      },
      nextStep() {
        this.step++;
      },
      submitApplication() {
        console.log(this.positionPoints)
        const formData = new FormData();
        for (const key in this.form) {
          formData.append(key, this.form[key]);
        }
        fetch('/api/installment-application', {
          method: 'POST',
          body: formData
        })
          .then(res => res.json())
          .then(res => alert('Submitted successfully'))
          .catch(err => alert('Error submitting application'));
      }
    }
  };
  </script>
  
  <style scoped>
  .v-card-title {
    background-color: #1976D2;
    color: white;
  }
  </style>
  