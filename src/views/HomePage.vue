<template>
  <ion-page>
    <ion-content :fullscreen="true">
      <div class="app-container">

        <!-- HEADER -->
        <header class="app-header">
          <div>
            <h1>My Pets 🐾</h1>
            <p>Your pets, all in a happy place</p>
          </div>

          <ion-button
            class="add-button"
            @click="openAddForm"
          >
            + Add Pet
          </ion-button>
        </header>

        <!-- SEARCH -->
        <div class="search-box">
          <ion-icon :icon="searchOutline" />

          <ion-input
            v-model="searchText"
            placeholder="Search pets..."
            :clear-input="true"
          />
        </div>

        <!-- PET COUNT -->
        <div class="pet-count">
          {{ filteredPets.length }}
          {{ filteredPets.length === 1 ? "pet" : "pets" }}
        </div>

        <!-- PET RECORDS -->
        <div
          v-if="filteredPets.length > 0"
          class="pet-list"
        >
          <div
            v-for="pet in filteredPets"
            :key="pet.id"
            class="pet-card"
            @click="openDetails(pet)"
          >
            <div class="pet-icon">
              {{ getPetIcon(pet.animalType) }}
            </div>

            <div class="pet-info">
              <h2>{{ pet.petName }}</h2>

              <p>
                {{ pet.animalType }}
                <span>•</span>
                {{ pet.age }}
                {{ pet.age === 1 ? "year" : "years" }}
              </p>

              <div
                class="health-status"
                :class="getStatusClass(pet)"
              >
                <span class="status-dot"></span>
                {{ getStatusText(pet) }}
              </div>
            </div>

            <!-- ACTION BUTTONS -->
            <div
              class="pet-actions"
              @click.stop
            >
              <ion-button
                fill="clear"
                class="view-action"
                @click="openDetails(pet)"
              >
                View
              </ion-button>

              <ion-button
                fill="clear"
                class="edit-action"
                @click="editPet(pet)"
              >
                Edit
              </ion-button>

              <ion-button
                fill="clear"
                class="delete-action"
                @click="confirmDelete(pet)"
              >
                Delete
              </ion-button>
            </div>
          </div>
        </div>

        <!-- EMPTY STATE -->
        <div
          v-else
          class="empty-state"
        >
          <div class="empty-icon">
            🐾
          </div>

          <h2>No pets found</h2>

          <p>
            Add your first pet to get started.
          </p>

          <ion-button
            class="empty-add-button"
            @click="openAddForm"
          >
            + Add Pet
          </ion-button>
        </div>

      </div>

      <!-- =========================
           ADD / EDIT PET MODAL
           ========================= -->

      <ion-modal
        :is-open="showForm"
        @didDismiss="closeForm"
      >
        <ion-page>

          <ion-header class="modal-header">
            <ion-toolbar>
              <ion-buttons slot="start">
                <ion-button @click="closeForm">
                  ‹
                </ion-button>
              </ion-buttons>

              <ion-title>
                {{ editingId ? "Edit Pet" : "Add New Pet" }}
              </ion-title>
            </ion-toolbar>
          </ion-header>

          <ion-content class="modal-content">

            <div class="form-container">

              <p class="form-subtitle">
                {{
                  editingId
                    ? "Update your pet's information"
                    : "Create a profile for your furry friend 💕"
                }}
              </p>

              <!-- PET NAME -->
              <div class="form-group">
                <ion-label>
                  Pet Name
                </ion-label>

                <ion-input
                  v-model="petForm.petName"
                  class="form-input"
                  fill="solid"
                  placeholder="e.g. Buddy"
                  type="text"
                />
              </div>

              <!-- ANIMAL TYPE -->
              <div class="form-group">
                <ion-label>
                  Animal Type
                </ion-label>

                <ion-input
                  v-model="petForm.animalType"
                  class="form-input"
                  fill="solid"
                  placeholder="Dog / Cat / Other"
                  type="text"
                />
              </div>

              <!-- BREED -->
              <div class="form-group">
                <ion-label>
                  Breed
                </ion-label>

                <ion-input
                  v-model="petForm.breed"
                  class="form-input"
                  fill="solid"
                  placeholder="e.g. Golden Retriever"
                  type="text"
                />
              </div>

              <!-- AGE -->
              <div class="form-group">
                <ion-label>
                  Age
                </ion-label>

                <ion-input
                  v-model.number="petForm.age"
                  class="form-input"
                  fill="solid"
                  type="number"
                  placeholder="e.g. 4"
                />
              </div>

              <!-- OWNER NAME -->
              <div class="form-group">
                <ion-label>
                  Owner Name
                </ion-label>

                <ion-input
                  v-model="petForm.ownerName"
                  class="form-input"
                  fill="solid"
                  placeholder="Your name"
                  type="text"
                />
              </div>

              <!-- NOTES -->
              <div class="form-group">
                <ion-label>
                  Notes
                </ion-label>

                <ion-textarea
                  v-model="petForm.notes"
                  class="form-textarea"
                  fill="solid"
                  placeholder="Anything important about your pet"
                  :auto-grow="true"
                />
              </div>

              <!-- SAVE / UPDATE -->
              <ion-button
                expand="block"
                class="save-button"
                @click="savePet"
              >
                {{ editingId ? "Update" : "Save Pet" }}
              </ion-button>

              <!-- CANCEL -->
              <ion-button
                v-if="editingId"
                expand="block"
                fill="clear"
                class="cancel-button"
                @click="closeForm"
              >
                Cancel
              </ion-button>

            </div>

          </ion-content>
        </ion-page>
      </ion-modal>

      <!-- =========================
           PET DETAILS MODAL
           ========================= -->

      <ion-modal
        :is-open="showDetails"
        @didDismiss="closeDetails"
      >
        <ion-page>

          <ion-header class="modal-header">
            <ion-toolbar>

              <ion-buttons slot="start">
                <ion-button @click="closeDetails">
                  ‹
                </ion-button>
              </ion-buttons>

              <ion-title>
                Pet Details
              </ion-title>

            </ion-toolbar>
          </ion-header>

          <ion-content class="modal-content">

            <div
              v-if="selectedPet"
              class="details-container"
            >

              <!-- PET SUMMARY -->
              <div class="details-card">

                <div class="details-icon">
                  {{ getPetIcon(selectedPet.animalType) }}
                </div>

                <div>
                  <h2>
                    {{ selectedPet.petName }}
                  </h2>

                  <p>
                    {{ selectedPet.animalType }}
                    •
                    {{ selectedPet.age }}
                    {{ selectedPet.age === 1 ? "year" : "years" }}
                  </p>

                  <div
                    class="health-status"
                    :class="getStatusClass(selectedPet)"
                  >
                    <span class="status-dot"></span>
                    {{ getStatusText(selectedPet) }}
                  </div>
                </div>

              </div>

              <!-- PET INFORMATION -->
              <div class="information-section">

                <div class="detail-row">
                  <span>Owner</span>
                  <strong>
                    {{ selectedPet.ownerName }}
                  </strong>
                </div>

                <div class="detail-row">
                  <span>Animal Type</span>
                  <strong>
                    {{ selectedPet.animalType }}
                  </strong>
                </div>

                <div class="detail-row">
                  <span>Breed</span>
                  <strong>
                    {{ selectedPet.breed }}
                  </strong>
                </div>

                <div class="detail-row">
                  <span>Age</span>
                  <strong>
                    {{ selectedPet.age }}
                    {{ selectedPet.age === 1 ? "year" : "years" }}
                  </strong>
                </div>

                <div class="detail-row notes-row">
                  <span>Notes</span>
                  <strong>
                    {{ selectedPet.notes || "No notes" }}
                  </strong>
                </div>

              </div>

              <!-- DETAILS BUTTONS -->
              <div class="details-actions">

                <ion-button
                  class="edit-details-button"
                  @click="editFromDetails"
                >
                  Edit Pet
                </ion-button>

                <ion-button
                  class="delete-details-button"
                  @click="confirmDelete(selectedPet)"
                >
                  Delete
                </ion-button>

              </div>

            </div>

          </ion-content>
        </ion-page>
      </ion-modal>

      <!-- =========================
           DELETE CONFIRMATION
           ========================= -->

      <ion-alert
        :is-open="showDeleteAlert"
        header="Are you sure?"
        :message="deleteMessage"
        :buttons="deleteButtons"
        @didDismiss="showDeleteAlert = false"
      />

    </ion-content>
  </ion-page>
</template>


<script setup lang="ts">

import {
  IonPage,
  IonContent,
  IonHeader,
  IonToolbar,
  IonTitle,
  IonButtons,
  IonButton,
  IonInput,
  IonTextarea,
  IonLabel,
  IonIcon,
  IonModal,
  IonAlert,
} from "@ionic/vue";

import {
  searchOutline,
} from "ionicons/icons";

import {
  ref,
  computed,
} from "vue";

import {
  ref as dbRef,
  push,
  set,
  onValue,
  update,
  remove,
} from "firebase/database";

import { db } from "@/firebase";


/* =========================
   PET TYPE
   ========================= */

interface Pet {
  id: string;
  petName: string;
  animalType: string;
  breed: string;
  age: number;
  ownerName: string;
  notes: string;
}


/* =========================
   PET FORM
   ========================= */

const petForm = ref({
  petName: "",
  animalType: "",
  breed: "",
  age: 0,
  ownerName: "",
  notes: "",
});


/* =========================
   PET RECORDS
   ========================= */

const pets = ref<Pet[]>([]);

const searchText = ref("");

const editingId = ref<string | null>(null);


/* =========================
   MODALS
   ========================= */

const showForm = ref(false);

const showDetails = ref(false);

const showDeleteAlert = ref(false);

const selectedPet = ref<Pet | null>(null);


/* =========================
   SEARCH
   ========================= */

const filteredPets = computed(() => {

  const search = searchText.value
    .toLowerCase()
    .trim();

  if (!search) {
    return pets.value;
  }

  return pets.value.filter((pet) =>

    pet.petName
      .toLowerCase()
      .includes(search)

    ||

    pet.animalType
      .toLowerCase()
      .includes(search)

    ||

    pet.breed
      .toLowerCase()
      .includes(search)

    ||

    pet.ownerName
      .toLowerCase()
      .includes(search)

  );

});


/* =========================
   CREATE / UPDATE
   ========================= */

const savePet = async () => {

  if (
    !petForm.value.petName ||
    !petForm.value.animalType ||
    !petForm.value.breed ||
    !petForm.value.ownerName
  ) {

    alert(
      "Please complete the required pet information."
    );

    return;
  }

  try {

    /* UPDATE */

    if (editingId.value) {

      const petRef = dbRef(
        db,
        `pets/${editingId.value}`
      );

      await update(
        petRef,
        {
          petName: petForm.value.petName,
          animalType: petForm.value.animalType,
          breed: petForm.value.breed,
          age: Number(petForm.value.age),
          ownerName: petForm.value.ownerName,
          notes: petForm.value.notes,
        }
      );

    }

    /* CREATE */

    else {

      const petsRef = dbRef(
        db,
        "pets"
      );

      const newPetRef = push(
        petsRef
      );

      await set(
        newPetRef,
        {
          petName: petForm.value.petName,
          animalType: petForm.value.animalType,
          breed: petForm.value.breed,
          age: Number(petForm.value.age),
          ownerName: petForm.value.ownerName,
          notes: petForm.value.notes,
        }
      );

    }

    clearForm();

    editingId.value = null;

    showForm.value = false;

  } catch (error) {

    console.error(
      "Error saving pet:",
      error
    );

    alert(
      "Unable to save pet."
    );

  }

};


/* =========================
   READ FROM FIREBASE
   ========================= */

const petsRef = dbRef(
  db,
  "pets"
);

onValue(
  petsRef,
  (snapshot) => {

    const data = snapshot.val();

    if (!data) {

      pets.value = [];

      return;
    }

    pets.value = Object.entries(data)

      .filter(
        ([, pet]) =>
          typeof pet === "object" &&
          pet !== null
      )

      .map(
        ([id, pet]) => {

          const record =
            pet as Record<string, unknown>;

          return {

            id,

            petName:
              String(
                record.petName ?? ""
              ),

            animalType:
              String(
                record.animalType ?? ""
              ),

            breed:
              String(
                record.breed ?? ""
              ),

            age:
              Number(
                record.age ?? 0
              ),

            ownerName:
              String(
                record.ownerName ?? ""
              ),

            notes:
              String(
                record.notes ?? ""
              ),

          };

        }
      );

  }
);


/* =========================
   EDIT PET
   ========================= */

const editPet = (pet: Pet) => {

  editingId.value = pet.id;

  petForm.value = {

    petName:
      pet.petName,

    animalType:
      pet.animalType,

    breed:
      pet.breed,

    age:
      pet.age,

    ownerName:
      pet.ownerName,

    notes:
      pet.notes,

  };

  showDetails.value = false;

  showForm.value = true;

};


/* =========================
   EDIT FROM DETAILS
   ========================= */

const editFromDetails = () => {

  if (!selectedPet.value) {
    return;
  }

  editPet(
    selectedPet.value
  );

};


/* =========================
   DELETE
   ========================= */

const deleteMessage = computed(() => {

  if (!selectedPet.value) {
    return "";
  }

  return `Delete ${selectedPet.value.petName}? This information will be permanently removed.`;

});


const confirmDelete = (pet: Pet) => {

  selectedPet.value = pet;

  showDeleteAlert.value = true;

};


const deleteButtons = [

  {
    text: "Cancel",
    role: "cancel",
  },

  {
    text: "Yes, Delete",
    role: "destructive",

    handler: async () => {

      if (!selectedPet.value) {
        return;
      }

      try {

        const petRef = dbRef(
          db,
          `pets/${selectedPet.value.id}`
        );

        await remove(
          petRef
        );

        showDetails.value = false;

        showDeleteAlert.value = false;

        selectedPet.value = null;

      } catch (error) {

        console.error(
          "Error deleting pet:",
          error
        );

        alert(
          "Unable to delete pet."
        );

      }

    },

  },

];


/* =========================
   ADD PET
   ========================= */

const openAddForm = () => {

  clearForm();

  editingId.value = null;

  showForm.value = true;

};


/* =========================
   PET DETAILS
   ========================= */

const openDetails = (pet: Pet) => {

  selectedPet.value = pet;

  showDetails.value = true;

};


const closeDetails = () => {

  showDetails.value = false;

  selectedPet.value = null;

};


/* =========================
   CLOSE FORM
   ========================= */

const closeForm = () => {

  showForm.value = false;

  editingId.value = null;

  clearForm();

};


/* =========================
   CLEAR FORM
   ========================= */

const clearForm = () => {

  petForm.value = {

    petName: "",
    animalType: "",
    breed: "",
    age: 0,
    ownerName: "",
    notes: "",

  };

};


/* =========================
   PET ICON
   ========================= */

const getPetIcon = (
  animalType: string
) => {

  const type =
    animalType.toLowerCase();

  if (
    type.includes("cat")
  ) {
    return "🐱";
  }

  if (
    type.includes("dog")
  ) {
    return "🐶";
  }

  if (
    type.includes("bird")
  ) {
    return "🐦";
  }

  if (
    type.includes("rabbit")
  ) {
    return "🐰";
  }

  return "🐾";

};


/* =========================
   HEALTH STATUS
   ========================= */

const getStatusText = (
  pet: Pet
) => {

  if (
    pet.notes
      .toLowerCase()
      .includes("check")
  ) {

    return "Needs check-up";

  }

  return "Healthy";

};


const getStatusClass = (
  pet: Pet
) => {

  if (
    pet.notes
      .toLowerCase()
      .includes("check")
  ) {

    return "status-warning";

  }

  return "status-healthy";

};

</script>


<style scoped>

/* =========================
   MAIN PAGE
   ========================= */

ion-content {
  --background: #fff7ed;
}

.app-container {
  min-height: 100vh;

  padding: 28px 18px 40px;

  background: #fff7ed;

  box-sizing: border-box;
}


/* =========================
   HEADER
   ========================= */

.app-header {

  display: flex;

  align-items: flex-start;

  justify-content: space-between;

  gap: 12px;

  margin-bottom: 22px;

}

.app-header h1 {

  margin: 0;

  color: #1f1a21;

  font-family: "Inter", sans-serif;

  font-size: 25px;

  font-weight: 700;

}

.app-header p {

  margin: 5px 0 0;

  color: #8c8291;

  font-family: "Inter", sans-serif;

  font-size: 11px;

}


/* =========================
   ADD PET BUTTON
   ========================= */

.add-button {

  --background: #ed7a63;

  --background-hover: #df6d59;

  --background-activated: #df6d59;

  --color: #ffffff;

  --border-radius: 10px;

  height: 40px;

  margin: 0;

  font-size: 11px;

  font-weight: 700;

  text-transform: none;

}


/* =========================
   SEARCH
   ========================= */

.search-box {

  display: flex;

  align-items: center;

  width: 100%;

  height: 44px;

  padding: 0 13px;

  background: #ffffff;

  border-radius: 12px;

  box-sizing: border-box;

  margin-bottom: 18px;

}

.search-box ion-icon {

  color: #9a909c;

  font-size: 16px;

  margin-right: 7px;

}

.search-box ion-input {

  --padding-start: 0;

  --padding-end: 0;

  --color: #1f1a21;

  --placeholder-color: #a69ca8;

  --placeholder-opacity: 1;

  font-size: 12px;

}


/* =========================
   PET COUNT
   ========================= */

.pet-count {

  margin-bottom: 10px;

  color: #1f1a21;

  font-size: 14px;

  font-weight: 700;

}


/* =========================
   PET LIST
   ========================= */

.pet-list {

  display: flex;

  flex-direction: column;

  gap: 12px;

}


/* =========================
   PET CARD
   ========================= */

.pet-card {

  position: relative;

  display: flex;

  align-items: flex-start;

  min-height: 116px;

  padding: 16px;

  padding-bottom: 48px;

  background: #ffffff;

  border-radius: 15px;

  box-sizing: border-box;

  cursor: pointer;

}

.pet-icon {

  display: flex;

  align-items: center;

  justify-content: center;

  width: 46px;

  height: 46px;

  margin-right: 12px;

  background: #e0d1fa;

  border-radius: 50%;

  font-size: 25px;

  flex-shrink: 0;

}

.pet-info {

  flex: 1;

  min-width: 0;

}

.pet-info h2 {

  margin: 0 0 5px;

  color: #1f1a21;

  font-size: 16px;

  font-weight: 700;

}

.pet-info p {

  margin: 0 0 8px;

  color: #8c8291;

  font-size: 11px;

}

.pet-info p span {

  margin: 0 3px;

}


/* =========================
   STATUS
   ========================= */

.health-status {

  display: flex;

  align-items: center;

  gap: 5px;

  font-size: 10px;

  font-weight: 600;

}

.status-dot {

  width: 7px;

  height: 7px;

  border-radius: 50%;

  display: inline-block;

}

.status-healthy {

  color: #3f9a6b;

}

.status-healthy .status-dot {

  background: #3f9a6b;

}

.status-warning {

  color: #ed7564;

}

.status-warning .status-dot {

  background: #ed7564;

}


/* =========================
   VIEW / EDIT / DELETE
   ========================= */

.pet-actions {

  display: flex;

  align-items: center;

  justify-content: flex-end;

  position: absolute;

  right: 10px;

  bottom: 8px;

}

.pet-actions ion-button {

  --padding-start: 9px;

  --padding-end: 9px;

  --padding-top: 4px;

  --padding-bottom: 4px;

  margin: 0;

  min-height: 32px;

  height: 32px;

  font-size: 11px;

  font-weight: 600;

  text-transform: none;

}

.view-action {

  --color: #9b7bd5;

}

.edit-action {

  --color: #3f9a6b;

}

.delete-action {

  --color: #d0444d;

}


/* =========================
   EMPTY STATE
   ========================= */

.empty-state {

  text-align: center;

  padding: 70px 20px;

}

.empty-icon {

  font-size: 44px;

  margin-bottom: 12px;

}

.empty-state h2 {

  margin: 0;

  color: #1f1a21;

  font-size: 20px;

}

.empty-state p {

  margin: 8px 0 20px;

  color: #8c8291;

  font-size: 12px;

}

.empty-add-button {

  --background: #ed7a63;

  --color: #ffffff;

  --border-radius: 12px;

  font-size: 12px;

  text-transform: none;

}


/* =========================
   MODAL HEADER
   ========================= */

.modal-header ion-toolbar {

  --background: #fff7ed;

  --color: #1f1a21;

  --border-width: 0;

}

.modal-header ion-title {

  color: #1f1a21;

  font-size: 17px;

  font-weight: 700;

}


/* =========================
   MODAL CONTENT
   ========================= */

.modal-content {

  --background: #fff7ed;

}


/* =========================
   FORM
   ========================= */

.form-container {

  padding: 20px 22px 40px;

}

.form-subtitle {

  margin: 0 0 24px;

  color: #6b5e70;

  font-size: 11px;

}


/* =========================
   FORM GROUP
   ========================= */

.form-group {

  margin-bottom: 17px;

}

.form-group ion-label {

  display: block;

  margin-bottom: 7px;

  color: #1f1a21;

  font-size: 11px;

  font-weight: 700;

}


/* =========================
   INPUT FIX
   ========================= */

.form-input {

  --background: #ffffff;

  --color: #1f1a21;

  --placeholder-color: #a69ca8;

  --placeholder-opacity: 1;

  --padding-start: 13px;

  --padding-end: 13px;

  --border-radius: 10px;

  --highlight-color-focused: #ed7a63;

  width: 100%;

  min-height: 46px;

  height: 46px;

  color: #1f1a21;

  background: #ffffff;

  border-radius: 10px;

  font-family: "Inter", sans-serif;

  font-size: 13px;

  box-sizing: border-box;

}


/* =========================
   TEXTAREA FIX
   ========================= */

.form-textarea {

  --background: #ffffff;

  --color: #1f1a21;

  --placeholder-color: #a69ca8;

  --placeholder-opacity: 1;

  --padding-start: 13px;

  --padding-end: 13px;

  --padding-top: 12px;

  --padding-bottom: 12px;

  --border-radius: 10px;

  --highlight-color-focused: #ed7a63;

  width: 100%;

  color: #1f1a21;

  background: #ffffff;

  border-radius: 10px;

  font-family: "Inter", sans-serif;

  font-size: 13px;

  box-sizing: border-box;

}


/* =========================
   SAVE BUTTON
   ========================= */

.save-button {

  --background: #ed7a63;

  --background-activated: #dc6d5a;

  --color: #ffffff;

  --border-radius: 10px;

  height: 47px;

  margin-top: 8px;

  font-size: 12px;

  font-weight: 700;

  text-transform: none;

}


/* =========================
   CANCEL BUTTON
   ========================= */

.cancel-button {

  --color: #8c8291;

  font-size: 11px;

  text-transform: none;

}


/* =========================
   DETAILS
   ========================= */

.details-container {

  padding: 20px 22px 40px;

}

.details-card {

  display: flex;

  align-items: center;

  padding: 18px;

  background: #e0d1fa;

  border-radius: 14px;

  margin-bottom: 26px;

}

.details-icon {

  display: flex;

  align-items: center;

  justify-content: center;

  width: 56px;

  height: 56px;

  margin-right: 14px;

  background: #ffffff;

  border-radius: 50%;

  font-size: 30px;

}

.details-card h2 {

  margin: 0 0 4px;

  color: #1f1a21;

  font-size: 18px;

  font-weight: 700;

}

.details-card p {

  margin: 0 0 7px;

  color: #6b5e70;

  font-size: 10px;

}


/* =========================
   INFORMATION
   ========================= */

.information-section {

  background: #ffffff;

  border-radius: 14px;

  padding: 5px 16px;

}

.detail-row {

  display: flex;

  flex-direction: column;

  padding: 13px 0;

  border-bottom: 1px solid #f0e9e4;

}

.detail-row:last-child {

  border-bottom: none;

}

.detail-row span {

  margin-bottom: 4px;

  color: #8c8291;

  font-size: 10px;

}

.detail-row strong {

  color: #1f1a21;

  font-size: 12px;

  font-weight: 700;

}

.notes-row strong {

  line-height: 1.5;

}


/* =========================
   DETAILS ACTIONS
   ========================= */

.details-actions {

  display: flex;

  gap: 10px;

  margin-top: 22px;

}

.details-actions ion-button {

  flex: 1;

  height: 44px;

  margin: 0;

  --border-radius: 10px;

  font-size: 11px;

  font-weight: 700;

  text-transform: none;

}

.edit-details-button {

  --background: #3f9a6b;

  --color: #ffffff;

}

.delete-details-button {

  --background: #d0444d;

  --color: #ffffff;

}


/* =========================
   DESKTOP
   ========================= */

@media (min-width: 600px) {

  .app-container {

    max-width: 500px;

    margin: 0 auto;

  }

}

</style>