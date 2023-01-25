<script setup >
  import { ref } from 'vue'

  const peso = ref(null)
  const port = ref(null)

  /* eslint-disable */

  const pesar = async() => {
    const writer = port.value.writable.getWriter();
    const data = new Uint8Array([80]);
    await writer.write(data);
    writer.releaseLock();
  }

  const conectar = async() => {

    const filters = [{ usbVendorId: 0x0483, usbProductId: 0x5740 }];
    port.value = await navigator.serial.requestPort({ filters });
    await port.value.open({baudRate: 9600});

    pesar();

    const textDecoder = new TextDecoderStream();
    const readableStreamClosed = port.value.readable.pipeTo(textDecoder.writable);

    while (textDecoder.readable){
      const reader = textDecoder.readable.getReader();
      try {
        while (true) {
          const { value, done } = await reader.read();
          console.log(await reader.read())
          if (done) {
            break;
          }else{
            console.log('value: ', value)
            peso.value = value

          }

        }
      } catch (error) {
        console.log('error')
      } finally {
        console.log('finally')
        reader.releaseLock();
      }
    }
    await textDecoder.close()
  }

  /* eslint-enable */

</script>

<template>
  <div>
    <h1>Bascula</h1>
    <button v-if="!port" @click="conectar()">Conectar</button>
    <!-- <button @click="pesar()">Peso</button> -->
    <button @click="pesar()">Enviar dato</button>
    <h2 v-if="peso">Peso: {{ peso }}</h2>
  </div>
</template>

<style scoped>
</style>