<script setup>
import { ref, computed } from 'vue';
import jsPDF from 'jspdf';
import html2canvas from 'html2canvas';

const currentDate = new Date().toLocaleDateString('en-GB');
const rows = ref([{ description: '', unit: '', qty: 0, sqty: 0, rate: 0, remarks: '' }]);
const editRows = ref([]);
const showEditModal = ref(false);

const openEditModal = () => {
  // Deep copy rows to editRows for editing
  editRows.value = rows.value.map(r => ({ ...r }));
  showEditModal.value = true;
};
const closeEditModal = () => {
  showEditModal.value = false;
};
const saveEditRows = () => {
  // Save edited rows to main table
  rows.value = editRows.value.map(r => ({ ...r }));
  showEditModal.value = false;
};
const addEditRow = () => {
  editRows.value.push({ description: '', unit: '', qty: 0, sqty: 0, rate: 0, remarks: '' });
};
const deleteEditRow = (idx) => {
  if (editRows.value.length > 1) editRows.value.splice(idx, 1);
};

const total = computed(() => rows.value.reduce((sum, r) => sum + r.sqty * r.rate, 0));
const ait = computed(() => (total.value * 5) / 100);
const grandTotal = computed(() => total.value + ait.value);

const savePDF = async () => {
  const el = document.getElementById('printTable');
  const canvas = await html2canvas(el);
  const img = canvas.toDataURL('image/png');
  const pdf = new jsPDF('p', 'mm', 'a4');
  const width = pdf.internal.pageSize.getWidth();
  const height = (canvas.height * width) / canvas.width;
  pdf.addImage(img, 'PNG', 0, 0, width, height);
  pdf.save('table.pdf');
};

const printTable = () => {
  window.print();
};
</script>

<template>
  <div class="p-6 max-w-5xl mx-auto bg-white shadow rounded-lg" id="printTable">
    <div class="flex flex-col md:flex-row md:justify-between mb-6">
      <div class="md:w-1/2 mb-4 md:mb-0">
        <div class="text-left">
          <div class="font-semibold">Ref, Subcontract No.DML5N-CP01-RSS-SCP-001 Addemdum No. 2c</div>
          <div>{{ currentDate }}</div>
          <div class="mt-2">Mr. Misawa Hideo</div>
          <div>Project Manager</div>
          <div>Constraction of Civil works and Land development at Depot MRT Line-5 N (CP-01)</div>
          <div>TOA Corporation</div>
          <div class="mt-2 font-bold">Interim Payment Certificate -03 (IPC-03)</div>
        </div>
      </div>
    </div>
    <h2 class="text-2xl font-bold mb-6 text-center text-gray-800">Material Delivery to Site</h2>

    <!-- Table + Totals -->

    <div class="overflow-x-auto">
      <table class="min-w-full border border-gray-300">
        <thead class="bg-gray-100">
          <tr>
            <th class="py-2 px-3 border-b text-center">Sl.No.</th>
            <th class="py-2 px-3 border-b text-center">Description</th>
            <th class="py-2 px-3 border-b text-center">Unit</th>
            <th class="py-2 px-3 border-b text-center">Schedule Quantity</th>
            <th class="py-2 px-3 border-b text-center">Supplied Quantity</th>
            <th class="py-2 px-3 border-b text-center">Rate (BDT)</th>
            <th class="py-2 px-3 border-b text-center">Amount (BDT)</th>
            <th class="py-2 px-3 border-b text-center">Remarks</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(r, i) in rows" :key="i" class="hover:bg-gray-50">
            <td class="py-2 px-3 text-center border-b">{{ i + 1 }}</td>
            <td class="py-2 px-3 border-b text-center">{{ r.description }}</td>
            <td class="py-2 px-3 border-b text-center">{{ r.unit }}</td>
            <td class="py-2 px-3 border-b text-center">{{ r.qty }}</td>
            <td class="py-2 px-3 border-b text-center">{{ r.sqty }}</td>
            <td class="py-2 px-3 border-b text-center">{{ r.rate }}</td>
            <td class="py-2 px-3 border-b text-center font-medium">{{ (r.sqty * r.rate).toFixed(2) }}</td>
            <td class="py-2 px-3 border-b text-center">{{ r.remarks }}</td>
          </tr>
        </tbody>
      </table>

      <!-- Totals -->
      <div class="mt-4 flex justify-end">
        <table class="border border-gray-300 w-1/2 text-right">
          <tbody>
            <tr class="bg-gray-50">
              <td class="py-2 px-4 font-semibold">Total (Excl. AIT)</td>
              <td class="py-2 px-4">{{ total.toFixed(2) }}</td>
            </tr>
            <tr>
              <td class="py-2 px-4 font-semibold">AIT (5%)</td>
              <td class="py-2 px-4">{{ ait.toFixed(2) }}</td>
            </tr>
            <tr class="bg-gray-50 font-bold">
              <td class="py-2 px-4">Grand Total</td>
              <td class="py-2 px-4">{{ grandTotal.toFixed(2) }}</td>
            </tr>
          </tbody>
        </table>
      </div>
      <!-- Bank Details -->
      <div class="flex justify-end mt-8">
        <div class="text-right bg-gray-50 p-4 rounded-lg shadow w-full max-w-md">
          <div class="font-bold mb-2">Bank Details</div>
          <div>Benificiery Name : <span class="font-semibold">RSS TRADE INTERNATIONAL</span></div>
          <div>BANK NAME : <span class="font-semibold">IFIC BANK LIMITED</span></div>
          <div>BRANCH : <span class="font-semibold">GABTALI</span></div>
          <div>A/C NO : <span class="font-semibold">1207094495001</span></div>
          <div>ROUNTING NO : <span class="font-semibold">120260120</span></div>
        </div>
      </div>
    </div>
  </div>

   <!-- Buttons -->
    <div class="mt-6 flex flex-wrap gap-3 justify-center mb-10">
      <button @click="openEditModal" class="bg-blue-500 hover:bg-blue-600 text-white font-semibold py-2 px-4 rounded shadow">
      ✏️ Edit Table
      </button>
      <button @click="savePDF" class="bg-green-500 hover:bg-green-600 text-white font-semibold py-2 px-4 rounded shadow">
      📥 Save PDF
      </button>
      <button @click="printTable" class="bg-gray-700 hover:bg-gray-800 text-white font-semibold py-2 px-4 rounded shadow">
      🖨 Print
      </button>
    </div>

  <!-- Edit Modal -->
  <div v-if="showEditModal" class="fixed inset-0 bg-black bg-opacity-40 flex items-center justify-center">
    <div class="bg-white rounded-lg shadow-lg p-6 w-[100vw] max-w-5xl relative">
      <h3 class="text-xl font-bold mb-4 text-center">Edit Table Data</h3>
      <div class="overflow-x-auto mb-4">
        <table class="min-w-full border border-gray-300">
          <thead class="bg-gray-100">
            <tr>
              <th class="py-2 px-3 border-b">Sl.No.</th>
              <th class="py-2 px-3 border-b">Description</th>
              <th class="py-2 px-3 border-b">Unit</th>
              <th class="py-2 px-3 border-b">Schedule Quantity</th>
              <th class="py-2 px-3 border-b">Supplied Quantity</th>
              <th class="py-2 px-3 border-b">Rate (BDT)</th>
              <th class="py-2 px-3 border-b">Amount (BDT)</th>
              <th class="py-2 px-3 border-b">Remarks</th>
              <th class="py-2 px-3 border-b">Delete</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(r, i) in editRows" :key="i" class="hover:bg-gray-50">
              <td class="py-2 px-3 text-center border-b">{{ i + 1 }}</td>
              <td class="py-2 px-3 border-b">
                <input v-model="r.description" class="w-full px-2 py-1 border rounded" />
              </td>
              <td class="py-2 px-3 border-b">
                <input v-model="r.unit" class="w-full px-2 py-1 border rounded" />
              </td>
              <td class="py-2 px-3 border-b text-right">
                <input v-model.number="r.qty" type="number" class="w-full px-2 py-1 border rounded text-right" />
              </td>
              <td class="py-2 px-3 border-b text-right">
                <input v-model.number="r.sqty" type="number" class="w-full px-2 py-1 border rounded text-right" />
              </td>
              <td class="py-2 px-3 border-b text-right">
                <input v-model.number="r.rate" type="number" class="w-full px-2 py-1 border rounded text-right" />
              </td>
              <td class="py-2 px-3 border-b text-right font-medium">{{ (r.sqty * r.rate).toFixed(2) }}</td>
              <td class="py-2 px-3 border-b">
                <input v-model="r.remarks" class="w-full px-2 py-1 border rounded" />
              </td>
              <td class="py-2 px-3 border-b text-center">
                <button @click="deleteEditRow(i)" class="text-red-500 hover:text-red-700 font-bold">🗑</button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
      <div class="flex gap-2 mb-4">
        <button @click="addEditRow" class="bg-blue-500 hover:bg-blue-600 text-white font-semibold py-2 px-4 rounded shadow">➕ Add Row</button>
      </div>
      <div class="flex gap-2 justify-end">
        <button @click="saveEditRows" class="bg-green-500 hover:bg-green-600 text-white font-semibold py-2 px-4 rounded shadow">Save</button>
        <button @click="closeEditModal" class="bg-gray-500 hover:bg-gray-600 text-white font-semibold py-2 px-4 rounded shadow">Cancel</button>
      </div>
    </div>
  </div>
</template>

<style>
/* Hide everything except the table when printing */
@media print {
  body * {
    visibility: hidden;
  }
  #printTable, #printTable * {
    visibility: visible;
  }
  #printTable {
    position: absolute;
    left: 0;
    top: 0;
    width: 100vw;
    max-width: 100vw;
    min-width: 0;
    box-sizing: border-box;
    overflow-x: visible;
    padding: 0 !important;
    margin: 0 !important;
  }
}
</style>
