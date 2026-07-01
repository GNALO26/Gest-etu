<script setup>
import { ref, reactive, computed, watch } from 'vue'

// --- État ---
const students = ref([
  { id: 1, nom: 'Dupont', prenom: 'Alice', filiere: 'Informatique', age: 20 },
  { id: 2, nom: 'Martin', prenom: 'Bob', filiere: 'Mathématiques', age: 22 },
  { id: 3, nom: 'Durand', prenom: 'Claire', filiere: 'Informatique', age: 19 },
])
let nextId = 4

const form = reactive({
  nom: '',
  prenom: '',
  filiere: '',
  age: ''
})
const editingId = ref(null) // null = ajout, sinon id de l'étudiant en cours de modif
const error = ref('')

// --- Propriétés calculées ---
const totalStudents = computed(() => students.value.length)

const averageAge = computed(() => {
  if (students.value.length === 0) return 0
  const sum = students.value.reduce((acc, s) => acc + s.age, 0)
  return (sum / students.value.length).toFixed(1)
})

// Récupérer les filières uniques avec leur effectif
const filieres = computed(() => {
  const map = new Map()
  students.value.forEach(s => {
    map.set(s.filiere, (map.get(s.filiere) || 0) + 1)
  })
  return Array.from(map, ([nom, count]) => ({ nom, count }))
})

// --- Méthodes ---
function resetForm() {
  form.nom = ''
  form.prenom = ''
  form.filiere = ''
  form.age = ''
  editingId.value = null
  error.value = ''
}

function editStudent(student) {
  form.nom = student.nom
  form.prenom = student.prenom
  form.filiere = student.filiere
  form.age = student.age.toString()
  editingId.value = student.id
  error.value = ''
}

function addOrUpdateStudent() {
  // Validation
  if (!form.nom.trim() || !form.prenom.trim() || !form.filiere.trim() || form.age === '') {
    error.value = 'Tous les champs sont requis.'
    return
  }
  const ageNum = parseInt(form.age, 10)
  if (isNaN(ageNum) || ageNum <= 0 || ageNum > 120) {
    error.value = 'Âge invalide (doit être un nombre entre 1 et 120).'
    return
  }

  if (editingId.value) {
    // Modification
    const index = students.value.findIndex(s => s.id === editingId.value)
    if (index !== -1) {
      students.value[index] = {
        ...students.value[index],
        nom: form.nom.trim(),
        prenom: form.prenom.trim(),
        filiere: form.filiere.trim(),
        age: ageNum
      }
    }
    editingId.value = null
  } else {
    // Ajout
    students.value.push({
      id: nextId++,
      nom: form.nom.trim(),
      prenom: form.prenom.trim(),
      filiere: form.filiere.trim(),
      age: ageNum
    })
  }
  resetForm()
}

function deleteStudent(id) {
  students.value = students.value.filter(s => s.id !== id)
  // Si on supprime l'étudiant en cours d'édition, on réinitialise le formulaire
  if (editingId.value === id) resetForm()
}

// --- Watchers ---
watch(() => form.age, (newVal) => {
  if (newVal !== '' && (isNaN(parseInt(newVal)) || parseInt(newVal) <= 0)) {
    error.value = 'Âge doit être un nombre positif.'
  } else {
    error.value = ''
  }
})
</script>

<template>
  <div class="dashboard">
    <!-- Sidebar gauche -->
    <aside class="sidebar">
      <h3>Tableau de bord</h3>
      <div class="stat-card">
        <span class="stat-label">Total étudiants</span>
        <span class="stat-value">{{ totalStudents }}</span>
      </div>
      
      <div class="stat-card">
        <span class="stat-label">Par filière</span>
        <ul v-if="filieres.length">
          <li v-for="f in filieres" :key="f.nom">
            {{ f.nom }} : {{ f.count }}
          </li>
        </ul>
        <p v-else class="text-muted">Aucune donnée</p>
      </div>
    </aside>

    <!-- Contenu principal -->
    <main class="main">
      <h2>Gestion des étudiants</h2>

      <!-- Formulaire -->
      <form @submit.prevent="addOrUpdateStudent" class="student-form">
        <div class="form-row">
          <div class="form-group">
            <label>Nom</label>
            <input v-model="form.nom" type="text" placeholder="Nom" />
          </div>
          <div class="form-group">
            <label>Prénom</label>
            <input v-model="form.prenom" type="text" placeholder="Prénom" />
          </div>
        </div>
        <div class="form-row">
          <div class="form-group">
            <label>Filière</label>
            <select v-model="form.filiere">
              <option value=""> Choisir </option>
              <option>Informatique</option>
              <option>Mathématiques</option>
              <option>Physique</option>
              <option>Biologie</option>
              <option>Lettres</option>
            </select>
          </div>
          <div class="form-group">
            <label>Âge</label>
            <input v-model="form.age" type="number" min="1" max="120" placeholder="Âge" />
          </div>
        </div>
        <p v-if="error" class="error-message">{{ error }}</p>
        <div class="form-actions">
          <button type="submit" class="btn-primary">
            {{ editingId ? 'Modifier' : 'Ajouter' }}
          </button>
          <button v-if="editingId" type="button" @click="resetForm" class="btn-secondary">Annuler</button>
        </div>
      </form>

      <!-- Liste des étudiants -->
      <div class="student-list">
        <h3>Liste des étudiants</h3>
        <div v-if="students.length === 0" class="empty">Aucun étudiant enregistré.</div>
        <table v-else class="table">
          <thead>
            <tr>
              <th>Nom</th>
              <th>Prénom</th>
              <th>Filière</th>
              <th>Âge</th>
              <th>Actions</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="student in students" :key="student.id">
              <td>{{ student.nom }}</td>
              <td>{{ student.prenom }}</td>
              <td>{{ student.filiere }}</td>
              <td>{{ student.age }}</td>
              <td class="actions-cell">
                <button @click="editStudent(student)" class="btn-icon" title="Modifier">✎</button>
                <button @click="deleteStudent(student.id)" class="btn-icon delete" title="Supprimer">✕</button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </main>
  </div>
</template>

<style scoped>
/* RESET & BASE */
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

.dashboard {
  display: flex;
  min-height: 100vh;
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  background: #f5f5f7;
  color: #1c1c1e;
}

/* SIDEBAR */
.sidebar {
  width: 260px;
  background: #ffffff;
  padding: 24px;
  border-right: 1px solid #e5e5ea;
  box-shadow: 2px 0 8px rgba(0,0,0,0.02);
}

.sidebar h3 {
  font-size: 1.2rem;
  font-weight: 600;
  margin-bottom: 24px;
  color: #1c1c1e;
}

.stat-card {
  background: #f9f9fb;
  border-radius: 12px;
  padding: 16px;
  margin-bottom: 16px;
}

.stat-label {
  display: block;
  font-size: 0.8rem;
  color: #8e8e93;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  margin-bottom: 6px;
}

.stat-value {
  font-size: 1.4rem;
  font-weight: 600;
}

.stat-card ul {
  list-style: none;
  padding: 0;
  margin-top: 8px;
}

.stat-card li {
  padding: 4px 0;
  font-size: 0.9rem;
  color: #3a3a3c;
}

.text-muted {
  color: #8e8e93;
  font-size: 0.85rem;
}

/* MAIN */
.main {
  flex: 1;
  padding: 32px;
  overflow-y: auto;
}

.main h2 {
  font-size: 1.6rem;
  font-weight: 600;
  margin-bottom: 24px;
}

/* FORM */
.student-form {
  background: white;
  border-radius: 16px;
  padding: 24px;
  margin-bottom: 32px;
  box-shadow: 0 4px 12px rgba(0,0,0,0.04);
}

.form-row {
  display: flex;
  gap: 16px;
  margin-bottom: 16px;
}

.form-group {
  flex: 1;
}

.form-group label {
  display: block;
  font-size: 0.85rem;
  font-weight: 500;
  margin-bottom: 4px;
  color: #3a3a3c;
}

.form-group input,
.form-group select {
  width: 100%;
  padding: 10px 12px;
  border: 1px solid #d1d1d6;
  border-radius: 10px;
  font-size: 0.95rem;
  background: #fafafa;
  transition: border-color 0.2s, box-shadow 0.2s;
}

.form-group input:focus,
.form-group select:focus {
  border-color: #007aff;
  box-shadow: 0 0 0 3px rgba(0,122,255,0.1);
  background: #fff;
}

.error-message {
  color: #ff3b30;
  font-size: 0.9rem;
  margin: -8px 0 12px;
}

.form-actions {
  display: flex;
  gap: 10px;
}

.btn-primary {
  padding: 10px 20px;
  background: #007aff;
  color: white;
  border: none;
  border-radius: 10px;
  font-weight: 500;
  cursor: pointer;
  transition: background 0.2s;
}

.btn-primary:hover {
  background: #0066d6;
}

.btn-secondary {
  padding: 10px 20px;
  background: #e5e5ea;
  color: #1c1c1e;
  border: none;
  border-radius: 10px;
  font-weight: 500;
  cursor: pointer;
  transition: background 0.2s;
}

.btn-secondary:hover {
  background: #d1d1d6;
}

/* TABLE */
.student-list {
  background: white;
  border-radius: 16px;
  padding: 24px;
  box-shadow: 0 4px 12px rgba(0,0,0,0.04);
}

.student-list h3 {
  font-size: 1.2rem;
  font-weight: 600;
  margin-bottom: 16px;
}

.table {
  width: 100%;
  border-collapse: collapse;
}

.table th,
.table td {
  text-align: left;
  padding: 12px 8px;
  border-bottom: 1px solid #f0f0f5;
}

.table th {
  font-size: 0.8rem;
  text-transform: uppercase;
  color: #8e8e93;
  font-weight: 600;
}

.table tbody tr:hover {
  background: #fafafa;
}

.actions-cell {
  display: flex;
  gap: 8px;
}

.btn-icon {
  background: none;
  border: 1px solid #d1d1d6;
  border-radius: 8px;
  width: 32px;
  height: 32px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  font-size: 0.9rem;
  color: #3a3a3c;
  transition: all 0.2s;
}

.btn-icon:hover {
  background: #f0f0f5;
  border-color: #007aff;
  color: #007aff;
}

.btn-icon.delete:hover {
  background: #ffeeed;
  border-color: #ff3b30;
  color: #ff3b30;
}

.empty {
  text-align: center;
  color: #8e8e93;
  padding: 24px;
}
</style>