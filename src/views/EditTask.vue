<script setup>
import { ref, onMounted } from "vue";
import { useRoute, useRouter } from "vue-router";
import { Preferences } from "@capacitor/preferences";
import { tasks} from '../state';
import {
  IonPage,
  IonHeader,
  IonToolbar,
  IonTitle,
  IonContent,
  IonList,
  IonItem,
  IonLabel,
  IonButton,
  IonInput,
} from "@ionic/vue";

const route = useRoute();
const router = useRouter();
const TASKS_KEY = "tasks";
const taskText = ref("");


// Cargar tarea seleccionada
const loadTask = async () => {
  const { value } = await Preferences.get({ key: TASKS_KEY });
  tasks.value = value ? JSON.parse(value) : [];

  // Obtener el índice de la tarea desde la URL
  const index = parseInt(route.params.index);
  taskText.value = tasks.value[index].text;
};

// Guardar cambios en la tarea editada 
const saveTask = async () => {
  const index = parseInt(route.params.index);
  tasks.value[index].text = taskText.value;
  await Preferences.set({ key: TASKS_KEY, value: JSON.stringify(tasks.value) });

  // Volver a la pantalla principal
  router.push("/");
};

// Volver al home
function goBack() {
  //router.push("/");
  router.go(-1);
};

// Cargar la tarea al montar el componente
onMounted(loadTask);

</script>

<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Editar Tarea</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content class="ion-padding">
      <h2>Editar tarea</h2>
      <ion-list>
        <!-- Campo de texto para editar la tarea -->
        <ion-item>
          <ion-input v-model="taskText"></ion-input>
        </ion-item>

        <ion-button @click="saveTask" size="small" color="success">
          Guardar cambios
        </ion-button>

        <ion-button @click="goBack()" size="small" color="medium">
          Regresar
        </ion-button>
      </ion-list>
    </ion-content>
  </ion-page>
</template>

<style scoped></style>