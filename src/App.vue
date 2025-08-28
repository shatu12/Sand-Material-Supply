<script setup>
import { ref, computed } from 'vue';
import jsPDF from 'jspdf';
import html2canvas from 'html2canvas';

const rows = ref([{ description: '', unit: '', qty: 0, sqty: 0, rate: 0, remarks: '' }]);
const addRow = () => rows.value.push({ description: '', unit: '', qty: 0, sqty: 0, rate: 0, remarks: '' });

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
    <h2 class="text-2xl font-bold mb-6 text-center text-gray-800">Material Delivery to Site</h2>

    <!-- Table + Totals -->
    <div class="overflow-x-auto">
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
          </tr>
        </thead>
        <tbody>
          <tr v-for="(r, i) in rows" :key="i" class="hover:bg-gray-50">
            <td class="py-2 px-3 text-center border-b">{{ i + 1 }}</td>
            <td class="py-2 px-3 border-b">{{ r.description }}</td>
            <td class="py-2 px-3 border-b">{{ r.unit }}</td>
            <td class="py-2 px-3 border-b text-right">{{ r.qty }}</td>
            <td class="py-2 px-3 border-b text-right">{{ r.sqty }}</td>
            <td class="py-2 px-3 border-b text-right">{{ r.rate }}</td>
            <td class="py-2 px-3 border-b text-right font-medium">{{ (r.qty * r.rate).toFixed(2) }}</td>
            <td class="py-2 px-3 border-b">{{ r.remarks }}</td>
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
    </div>
  </div>

   <!-- Buttons -->
    <div class="mt-6 flex flex-wrap gap-3 justify-center">
      <button @click="addRow" class="bg-blue-500 hover:bg-blue-600 text-white font-semibold py-2 px-4 rounded shadow">
        ➕ Add Row
      </button>
      <button @click="savePDF" class="bg-green-500 hover:bg-green-600 text-white font-semibold py-2 px-4 rounded shadow">
        📥 Save PDF
      </button>
      <button @click="printTable" class="bg-gray-700 hover:bg-gray-800 text-white font-semibold py-2 px-4 rounded shadow">
        🖨 Print
      </button>
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
    width: 100%;
  }
}
</style>
