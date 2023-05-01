<script setup lang="ts">
import {ref} from "vue";
import axios from "axios";

const isSubmitting = ref(true);
const fileGenerated = ref()

const fileInput = ref<HTMLInputElement | null>(null);
const submitForm = async () => {
    const file = fileInput.value?.files?.[0];
    if (!file) {
        console.error('Aucun fichier sélectionné');
        return;
    }

    const formData = new FormData();
    formData.append('fileInput', file);

    try {
        const response = await axios.post('http://localhost:3000/upload', formData);

        //let characterCount = 0;
        //console.log(`Nombre de caractères : ${characterCount}`);
        //const estimatedPrice = (characterCount * 0.0004).toFixed(2)
        //console.log(`Prix estimé : ${estimatedPrice}`)

        axios.post('http://localhost:3000/translate/document', {
            document: response.data,
            lang: "fr",
            targetLang: "en-GB"
        }).then(response => {
            console.log(response.data); // Affiche le texte traduit
            fileGenerated.value = response.data
        }).catch(error => {
            console.error(error);
        });


    } catch (error) {
        console.error('Erreur lors de la requête');
    } finally {
        isSubmitting.value = false;
    }
};

function saveJSON() {
    const jsonData = JSON.stringify(fileGenerated.value)
    const blob = new Blob([jsonData], {type: 'application/json'});
    const url = URL.createObjectURL(blob);
    const link = document.createElement('a');
    link.download = 'myFile.json';
    link.href = url;
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
}

</script>

<template>
    <input type="file" ref="fileInput">
    <button type="submit" @click="submitForm">Envoyer</button>
    <button @click="saveJSON" :disabled="isSubmitting">Enregistrer JSON</button>
</template>

<style scoped>
.logo {
    height: 6em;
    padding: 1.5em;
    will-change: filter;
    transition: filter 300ms;
}

.logo:hover {
    filter: drop-shadow(0 0 2em #646cffaa);
}

.logo.vue:hover {
    filter: drop-shadow(0 0 2em #42b883aa);
}
</style>
