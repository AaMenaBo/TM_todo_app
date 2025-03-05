<script setup>
import { ref, onMounted } from "vue";
import { tasks } from '../state';
import { Preferences } from "@capacitor/preferences";
import { alertController } from '@ionic/vue';
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

// En este proyecto decidí dar comentario a todo el codigo para poder tener una idea clara

// Clave para almacenar tareas
const TASKS_KEY = "tasks";

// Texto 
const newTaskText = ref("");

//  Función para cargar tareas almacenadas
const loadTasks = async () => {
  const { value } = await Preferences.get({ key: TASKS_KEY });
  tasks.value = value ? JSON.parse(value) : [];
};

//  Función para guardar las tareas en el almacenamiento local
const saveTasks = async () => {
  await Preferences.set({ key: TASKS_KEY, value: JSON.stringify(tasks.value) });
};

//  Función para agregar una nueva tarea
const addNewTask = async () => {
  if (newTaskText.value.trim()) {
    // Crear nueva tarea
    const task = { text: newTaskText.value, completed: false };

    // Agregarla al array
    tasks.value.push(task);

    // Guardar en almacenamiento local
    await saveTasks();

    // Limpiar input
    newTaskText.value = "";
  }
};

//  Función para marcar una tarea como completada o no completada
const toggleCompletion = async (index) => {

  // Cambiar el estado de la tarea
  tasks.value[index].completed = !tasks.value[index].completed;

  // Guardar cambios
  await saveTasks();
};

// Función para eliminar una tarea por índice
const deleteTask = async (index) => {

  // Remover tarea del array
  tasks.value.splice(index, 1);

  // Guardar cambios
  await saveTasks();
};

// Función para mostrar el alert de confirmación
const showDeleteAlert = async (index) => {
  const alert = await alertController.create({
    header: 'Borrar tarea',
    subHeader: '¿Estás seguro de que deseas borrar esta tarea?',
    message: 'Esta acción no se puede deshacer.',
    buttons: [
      {
        text: 'Cancelar',
        role: 'cancel',
      },
      {
        text: 'Borrar',
        role: 'destructive',
        handler: () => {
          deleteTask(index);
        },
      },
    ],
  });

  await alert.present();
};

//  Cargar tareas al montar el componente
onMounted(loadTasks);
</script>

<template>
  <ion-page>
    <ion-header :translucent="true">
      <ion-toolbar>
        <ion-buttons slot="start">
          <ion-menu-button color="primary"></ion-menu-button>
        </ion-buttons>
        <ion-title>Home Task</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content>
      <!-- Lista de Tareas -->
      <ion-list>
        <ion-item v-for="(task, index) in tasks" :key="index">
          <!-- Nombre de la tarea -->
          <ion-label :class="{ completed: task.completed }">
            {{ task.text }}
          </ion-label>

          <!-- Botón para marcar como completada -->
          <ion-button @click="toggleCompletion(index)" color="success" fill="clear">Completar</ion-button>

          <!-- Botón para ir a Editar Tarea-->
          <ion-button :router-link="'/edit/' + index" color="medium" fill="clear">Editar</ion-button>

          <!-- Botón para eliminar la tarea -->
          <ion-button @click="showDeleteAlert(index)" fill="solid" color="danger">Borrar</ion-button>
        </ion-item>
      </ion-list>

      <!-- Input para agregar nueva tarea -->
      <ion-item>
        <ion-input v-model="newTaskText" placeholder="Nueva tarea"></ion-input>
        <ion-button @click="addNewTask" color="primary" fill="solid">Agregar</ion-button>
      </ion-item>
    </ion-content>
  </ion-page>
</template>

<style scoped>
/* Estilo para tareas completadas */
.completed {
  text-decoration: line-through;
  color: gray;
}
</style>
