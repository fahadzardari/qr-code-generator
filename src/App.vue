<template>

<div class="container">
      <form class="input-field" @submit.prevent="showUrl">
        <input type="text"  v-model="storedUrl">
        <button type="submit">Generate</button>
      </form>
  <div v-show="generated" class="qr">
    <qr id="qrCode" ref="generatedQr" :value="url" :size=200 level="H"/>
    <button @click="downloadQr" class="download-button">Download</button>
  </div>
  </div>
</template>
<script>
  import qr from 'qrcode.vue';
  import html2canvas from 'html2canvas';

export default {
  components: {
    qr,
  },
  data() {
    return {
      url: '1',
      generated: false,
      storedUrl: ''
    }
  },
  methods: {
    showUrl() {

      if (this.storedUrl.length > 0) {

        this.generated = true;
        this.url = this.storedUrl;
      }
    },
    downloadQr(){
      // const image = document.querySelectorAll("qrCode");
      const image = document.querySelectorAll("canvas")[0];
      console.log(image)
      html2canvas(image)
        .then((canvas)=>{
         //   alert(canvas)
            const base64Image = canvas.toDataURL("image/png");
            var a = document.createElement('a');
            a.setAttribute("href" , base64Image);
            a.setAttribute("download" , this.url + ".png");
            a.click();
            a.remove();
        });
    }

  },
  
}
</script>


<style >
  .container {
    display: flex;
    flex-direction: column;
    gap: 5rem;
    justify-items: center;
    align-items: center;
  }
  
  
  .input-field > input {
    width: 15rem;
    border-radius: 0;
    padding: 10px;
  }
  .input-field {
    display: flex;
    flex-direction: row;
    gap: 0;
  }
  
  .input-field>button {
    background-color: navy;
    color: white;
    border-radius: 0;
  }
  .qr{
    display: flex;
    flex-direction: column;
    gap: 2rem;
  }
  .download-button{
    color:white;
    background-color: navy;
  }
  </style>