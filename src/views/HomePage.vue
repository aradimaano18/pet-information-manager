<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Pet Information Manager</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content class="ion-padding">

      <ion-card>
        <ion-card-header>
          <ion-card-title>Add Pet</ion-card-title>
        </ion-card-header>

        <ion-card-content>

          <ion-item>
            <ion-input
              v-model="petForm.petName"
              label="Pet Name"
              label-placement="floating"
              placeholder="Enter pet name"
          ></ion-input>  
          </ion-item>

          <ion-item>
            <ion-input
              v-model="petForm.animalType"
              label="Animal Type"
              label-placement="floating"
              placeholder="Enter animal type"
          ></ion-input>
          </ion-item>

          <ion-item>
            <ion-input
              v-model="petForm.breed"
              label="Breed"
              label-placement="floating"
              placeholder="Enter breed"
          ></ion-input>
          </ion-item>

          <ion-item>
            <ion-input
              v-model.number="petForm.age"
              type="number"
              label="Age"
              label-placement="floating"
              placeholder="Enter age"
          ></ion-input>
          </ion-item>

          <ion-item>
            <ion-input
              v-model="petForm.ownerName"
              label="Owner Name"
              label-placement="floating"
              placeholder="Enter owner name"
          ></ion-input>   
          </ion-item>

          <ion-item>
            <ion-textarea
              v-model="petForm.notes"
              label="Notes"
              label-placement="floating"
              placeholder="Enter notes"
          ></ion-textarea>
          </ion-item>

          <ion-button expand="block" @click="addPet">
            {{ editingId ? "Update Pet" : "Add Pet" }}
          </ion-button>

        </ion-card-content>
      </ion-card>

      <ion-card>
        <ion-card-header>
          <ion-card-title>Pet Records</ion-card-title>
        </ion-card-header>

        <ion-card-content>
          <ion-list>
            <ion-item v-for="pet in pets" :key="pet.id">
              <ion-label>
                <h2>{{ pet.petName }}</h2>
                <p>{{ pet.animalType }} - {{ pet.breed }}</p>
                <p>Age: {{ pet.age }}</p>
                <p>Owner: {{ pet.ownerName }}</p>
                <p>Notes: {{ pet.notes }}</p>
              </ion-label>

              <ion-button fill="clear" @click="editPet(pet)">
                Edit
              </ion-button>

              <ion-button fill="clear" color="danger" @click="deletePet(pet.id)">
                Delete
              </ion-button>
            </ion-item>
          </ion-list>
        </ion-card-content>
      </ion-card>

    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import {
  IonPage,
  IonHeader,
  IonToolbar,
  IonTitle,
  IonContent,
  IonCard,
  IonCardHeader,
  IonCardTitle,
  IonCardContent,
  IonItem,
  IonInput,
  IonTextarea,
  IonButton,
  IonList,
  IonLabel
} from "@ionic/vue";

import { ref } from "vue";
import { db } from "@/firebase";
import { ref as dbRef, push, set, onValue, update, remove } from "firebase/database";
const petForm = ref({
  petName: "",
  animalType: "",
  breed: "",
  age: 0,
  ownerName: "",
  notes: ""
});

const pets = ref<any[]>([]);
const editingId = ref<string | null>(null);

const editPet = (pet: any) => {
  editingId.value = pet.id;

  petForm.value = {
    petName: pet.petName,
    animalType: pet.animalType,
    breed: pet.breed,
    age: pet.age,
    ownerName: pet.ownerName,
    notes: pet.notes
  };
};

const addPet = async () => {
  if (editingId.value) {
    const petRef = dbRef(db, `pets/${editingId.value}`);

    await update(petRef, {
      petName: petForm.value.petName,
      animalType: petForm.value.animalType,
      breed: petForm.value.breed,
      age: petForm.value.age,
      ownerName: petForm.value.ownerName,
      notes: petForm.value.notes
    });

    editingId.value = null;
  } else {
    const petsRef = dbRef(db, "pets");
    const newPetRef = push(petsRef);

    await set(newPetRef, {
      petName: petForm.value.petName,
      animalType: petForm.value.animalType,
      breed: petForm.value.breed,
      age: petForm.value.age,
      ownerName: petForm.value.ownerName,
      notes: petForm.value.notes
    });
  }

   clearForm();
};


const clearForm = () => {
  petForm.value = {
    petName: "",
    animalType: "",
    breed: "",
    age: 0,
    ownerName: "",
    notes: ""
  };
};

const deletePet = async (id: string) => {
  const confirmed = confirm("Are you sure you want to delete this pet?");

  if (!confirmed) {
    return;
  }

  const petRef = dbRef(db, `pets/${id}`);
  await remove(petRef);
};

const petsRef = dbRef(db, "pets");

onValue(petsRef, (snapshot) => {
  const data = snapshot.val();

  if (data) {
   pets.value = Object.entries(data)
  .filter(([_, pet]) => typeof pet === "object" && pet !== null)
  .map(([id, pet]) => ({
    id,
    ...(pet as object)
  }));
  } else {
    pets.value = [];
  }
});
</script>