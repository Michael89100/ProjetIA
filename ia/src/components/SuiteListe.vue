<template>
  <v-container style="max-width: 1200px; padding-top: 20px;">
    <v-row>
      <!-- Liste des suites à gauche -->
      <v-col cols="12" md="8" class="suites-list">
        <h2 class="text-h4 font-weight-bold mb-4">Liste des Suites</h2>
        <v-data-table :headers="headers" :items="suites" class="elevation-2">
          <template v-slot:[`item.action`]="{ item }">
            <v-btn icon small class="mr-2" @click="editSuite(item)" style="color: darkorange;">
              <v-icon small>mdi-pencil</v-icon>
            </v-btn>
            <v-btn icon small class="mr-2" @click="deleteSuite(item.id)" style="color: darkred;">
              <v-icon small>mdi-delete</v-icon>
            </v-btn>
          </template>
        </v-data-table>
      </v-col>

      <!-- Formulaire à droite -->
      <v-col cols="12" md="4" class="form-section">
        <h2 class="text-h5 font-weight-bold">{{ editIndex === -1 ? 'Ajouter une Suite' : 'Modifier la Suite' }}</h2>
        <v-form @submit.prevent="addSuite">
          <v-text-field v-model="newSuite.nbRooms" :rules="roomRules" label="Nombre de chambres" required></v-text-field>
          <v-text-field v-model="newSuite.surface" :rules="surfaceRules" label="Surface (m²)" required></v-text-field>
          <v-text-field v-model="newSuite.nbFenetre" :rules="windowRules" label="Nombre de fenêtres" required></v-text-field>
          <v-text-field v-model="newSuite.price" :rules="priceRules" label="Prix (€)" required></v-text-field>
          <v-date-picker v-model="newSuite.annee" :rules="anneeRules" label="Année" required></v-date-picker>
          <v-select
            class="mt-2"
            v-model="newSuite.ville"
            :items="availableCities"
            label="Ville"
            required
          ></v-select>
          <v-checkbox v-model="newSuite.balcon" label="Balcon"></v-checkbox>
          <v-checkbox v-model="newSuite.garage" label="Garage"></v-checkbox>
          <v-row>
            <v-col cols="12" class="d-flex align-center">
              <label>Note</label>
              <v-rating v-model="newSuite.note" :rules="noteRules" required></v-rating>
            </v-col>
          </v-row>
          <v-select v-model="newSuite.price_category" :items="priceCategories" label="Catégorie de prix" required></v-select>
          <v-btn
            class="gradient-btn mt-4"
            dark
            type="submit"
            block
          >
            {{ editIndex === -1 ? 'Ajouter et Sauvegarder' : 'Modifier et Sauvegarder' }}
          </v-btn>
        </v-form>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
export default {
  name: 'SuiteListe',
  data() {
    return {
      suites: [],
      newSuite: { nbRooms: '', surface: '', nbFenetre: '', price: '', annee: null, balcon: false, garage: false, note: 0, price_category: '', ville: '' },
      editIndex: -1,
      headers: [
        { title: 'ID', value: 'id' },
        { title: 'Chambres', value: 'nbRooms' },
        { title: 'Surface (m²)', value: 'surface' },
        { title: 'Fenêtres', value: 'nbFenetre' },
        { title: 'Prix (€)', value: 'price' },
        { title: 'Année', value: 'annee' },
        { title: 'Balcon', value: 'balcon' },
        { title: 'Garage', value: 'garage' },
        { title: 'Note', value: 'note' },
        { title: 'Catégorie de prix', value: 'price_category' },
        { title: 'Ville', value: 'ville' },
        { title: 'Actions', value: 'action', sortable: false },
      ],
      roomRules: [
        v => (!v || v.length === 0 || (v >= 1 && v <= 100)) || 'Le nombre de chambres doit être entre 1 et 100'
      ],
      surfaceRules: [
        v => (!v || v.length === 0 || (v >= 5 && v <= 600)) || 'La surface doit être entre 5 et 600 m²'
      ],
      windowRules: [
        v => (!v || v.length === 0 || (v >= 1 && v <= 100)) || 'Le nombre de fenêtres doit être entre 1 et 100'
      ],
      priceRules: [
        v => (!v || v.length === 0 || (v >= 50000 && v <= 1000000)) || 'Le prix doit être entre 50,000 et 1,000,000'
      ],
      anneeRules: [
        v => (!v || v.length === 0 || (v >= new Date(2004, 0, 1) && v <= new Date(2024, 0, 1))) || 'L\'année doit être entre 2004 et 2024'
      ],
      noteRules: [
        v => (!v || v.length === 0 || (v >= 1 && v <= 5)) || 'La note doit être entre 1 et 5'
      ],
      priceCategories: ['low', 'normal', 'high', 'scam'],
      availableCities: ['Paris', 'Lyon', 'Marseille']
    };
  },
  mounted() {
    // Récupérer les données via l'API
    fetch('http://localhost:3000/suites')
      .then(response => response.json())
      .then(data => {
        this.suites = data;
        console.log(this.suites);
      })
      .catch(error => {
        console.error('Erreur lors de la récupération des données :', error);
      });
  },
  methods: {
    addSuite() {
      const isNew = this.editIndex === -1;
      const suiteToAdd = { ...this.newSuite, id: isNew ? this.suites.length + 1 : this.suites[this.editIndex].id };
      if (isNew) {
        this.suites.push(suiteToAdd);
      } else {
        Object.assign(this.suites[this.editIndex], this.newSuite);
      }
      this.saveData(isNew);
    },

    saveData(isNew) {
      fetch('http://localhost:3000/save', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({ suites: this.suites }),
      })
        .then((response) => response.json())
        .then((data) => {
          alert('Données sauvegardées avec succès: ' + data.filePath);
          this.resetForm();  // Réinitialiser le formulaire après succès
          if (!isNew) {
            this.editIndex = -1;  // Réinitialiser l'index d'édition
          }
        })
        .catch((error) => {
          console.error('Erreur lors de la sauvegarde des données:', error);
        });
    },

    resetForm() {
      this.newSuite = { nbRooms: '', surface: '', nbFenetre: '', price: '', annee: null, balcon: false, garage: false, note: 0, price_category: '', ville: '' };
    },
    editSuite(suite) {
      this.newSuite = { ...suite };
      this.editIndex = this.suites.findIndex((s) => s.id === suite.id);
    },
    deleteSuite(id) {
      this.suites = this.suites.filter((suite) => suite.id !== id);
    },
  },
};
</script>

<style scoped>
.suites-list {
  padding: 20px;
  background-color: #ffffff;
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
}

.form-section {
  padding: 20px;
  background-color: #f5f5f5;
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
}

h2 {
  margin-bottom: 20px;
}

.v-data-table {
  margin-bottom: 20px;
}

.gradient-btn {
  background: linear-gradient(to right, #00c6ff, #0072ff);
  color: white;
  font-weight: bold;
  border-radius: 8px;
  padding: 10px 20px;
  transition: all 0.3s ease;
}

.gradient-btn:hover {
  background: linear-gradient(to right, #0072ff, #00c6ff);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

.v-btn {
  margin-top: 10px;
}

.v-icon {
  cursor: pointer;
  transition: color 0.2s ease;
}

.v-icon:hover {
  color: #ff5722;
}

label {
  margin-right: 8px;
}
</style>
