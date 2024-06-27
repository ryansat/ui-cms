<template>
    <button class="export-button" @click="exportToPDF">Export to PDF</button>
  </template>
  
  <script>
  import html2pdf from 'html2pdf.js'; 
  
  export default {
    methods: {
      exportToPDF() {
        const paperElement = this.$parent.$refs.paper; // Get a reference to the paper container element
  
        // Configure the pdfOptions as needed (e.g., page size, orientation, etc.)
        const pdfOptions = {
          margin: 10,
          filename: 'exported-paper.pdf',
          image: { type: 'jpeg', quality: 0.98 },
          html2canvas: { scale: 2 },
          jsPDF: { unit: 'mm', format: 'a4', orientation: 'portrait' },
        };
  
        // Use html2pdf to generate the PDF from the paper element
        html2pdf()
          .from(paperElement)
          .set(pdfOptions)
          .outputPdf()
          .then((pdf) => {
            const blob = new Blob([pdf], { type: 'application/pdf' });
            const url = window.URL.createObjectURL(blob);
            window.open(url, '_blank');
          });
      },
    },
  };
  </script>
  
  <style scoped>
  .export-button {
    background-color: #007bff;
    color: #fff;
    border: none;
    padding: 10px 20px;
    cursor: pointer;
  }
  </style>
  