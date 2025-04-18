<script setup>
import { ref, onMounted, watch, computed } from 'vue'
import { useTheme } from 'vuetify'

const theme = useTheme()
const currentTheme = ref(localStorage.getItem('theme') || 'light')

const dialog = ref(false)
const loading = ref(false)
const snackbar = ref(false)
const snackbarMsg = ref('')
const isEditing = ref(false)

const profileImage = ref('')
const selectedFile = ref(null)
const maxSizeMB = 2

const searchQuery = ref('')

function performSearch() {
  if (searchQuery.value.trim()) {
    snackbarMsg.value = `You searched for: "${searchQuery.value}"`
    snackbar.value = true
  }
}

const profile = ref({
  firstName: '',
  lastName: '',
  middleInitial: '',
  age: '',
  expertise: '',
  email: '',
  phone: '',
  about: '',
  education: ['', '', ''],
})

function toggleTheme() {
  currentTheme.value = currentTheme.value === 'light' ? 'dark' : 'light'
  theme.global.name.value = currentTheme.value
  localStorage.setItem('theme', currentTheme.value)
}

watch(currentTheme, (val) => {
  theme.global.name.value = val
  localStorage.setItem('theme', val)
})

onMounted(() => {
  theme.global.name.value = currentTheme.value
  const storedImage = localStorage.getItem('profileImage')
  if (storedImage) profileImage.value = storedImage
})

watch(profileImage, (newVal) => {
  if (newVal) localStorage.setItem('profileImage', newVal)
})

function proceedWithApplication() {
  loading.value = true
  setTimeout(() => {
    loading.value = false
    dialog.value = false
    snackbarMsg.value = 'Application submitted successfully!'
    snackbar.value = true
  }, 2000)
}

function enableEdit() {
  isEditing.value = true
}
function cancelEdit() {
  isEditing.value = false
}
function saveProfile() {
  isEditing.value = false
  snackbarMsg.value = 'Profile saved!'
  snackbar.value = true
}

function onImageSelected(event) {
  const file = event.target.files[0]
  if (!file) return

  const allowedTypes = ['image/jpeg', 'image/png']
  if (!allowedTypes.includes(file.type)) {
    snackbarMsg.value = 'Only JPG and PNG files are allowed.'
    snackbar.value = true
    return
  }

  const sizeMB = file.size / 1024 / 1024
  if (sizeMB > maxSizeMB) {
    snackbarMsg.value = `File size exceeds ${maxSizeMB}MB limit.`
    snackbar.value = true
    return
  }

  selectedFile.value = file

  const reader = new FileReader()
  reader.onload = (e) => {
    profileImage.value = e.target.result
  }
  reader.readAsDataURL(file)
}

function removeProfileImage() {
  profileImage.value = ''
  localStorage.removeItem('profileImage')
}

const fullName = computed(() => {
  return `${profile.value.firstName} ${profile.value.middleInitial}. ${profile.value.lastName}`
})

function getEducationPlaceholder(index) {
  if (index === 0) return 'Enter your school/university'
  if (index === 1) return 'Enter your course or degree'
  if (index === 2) return 'Enter your year level'
  return 'Enter educational info'
}
</script>
<template>
  <v-app id="inspire">
    <!-- App Bar -->
    <v-app-bar
      flat
      :color="currentTheme === 'light' ? '#1565c0' : 'grey-darken-4'"
      class="px-2 px-md-4"
    >
      <v-container fluid class="d-flex align-center justify-space-between pa-0">
        <!-- Logo -->
        <v-avatar color="#fff" size="44" class="mr-2">
          <v-img src="image/Teach&Learn.png" alt="Logo" />
        </v-avatar>

        <!-- Desktop Navigation -->
        <v-spacer />

        <!-- Centered Navigation Links (Desktop only) -->
        <div class="d-none d-md-flex align-center" style="gap: 24px">
          <RouterLink to="/home" class="text-white text-decoration-none font-weight-medium"
            >Home</RouterLink
          >
          <RouterLink to="/about" class="text-white text-decoration-none font-weight-medium"
            >About Us</RouterLink
          >
          <RouterLink to="/contact" class="text-white text-decoration-none font-weight-medium"
            >Contact Us</RouterLink
          >
        </div>

        <!-- Spacer after center links -->
        <v-spacer />

        <!-- Mobile Search and Menu -->
        <div class="d-flex align-center gap-2">
          <!-- Search -->
          <v-text-field
            v-model="searchQuery"
            placeholder="Search..."
            density="compact"
            flat
            rounded="lg"
            hide-details
            class="search-input d-none d-sm-flex"
            style="max-width: 160px"
            append-inner-icon="mdi-magnify"
            @keydown.enter="performSearch"
            @click:append-inner="performSearch"
          />

          <!-- Mobile Menu Button -->
          <v-menu transition="scale-transition" offset-y>
            <template #activator="{ props }">
              <v-app-bar-nav-icon v-bind="props" class="d-md-none" />
            </template>
            <v-list>
              <v-list-item>
                <router-link to="/home" class="text-decoration-none">Home</router-link>
              </v-list-item>
              <v-list-item>
                <router-link to="/profile" class="text-decoration-none">My Profile</router-link>
              </v-list-item>
              <v-list-item>
                <router-link to="/appointments" class="text-decoration-none"
                  >My Appointments</router-link
                >
              </v-list-item>
              <v-list-item>
                <router-link to="/about" class="text-decoration-none">About Us</router-link>
              </v-list-item>
              <v-list-item>
                <router-link to="/contact" class="text-decoration-none">Contact Us</router-link>
              </v-list-item>
              <v-divider></v-divider>
              <v-list-item>
                <router-link to="/logout" class="text-decoration-none">Logout</router-link>
              </v-list-item>
            </v-list>
          </v-menu>
        </div>
      </v-container>
    </v-app-bar>

    <!-- Main Content -->
    <v-main :class="currentTheme === 'dark' ? 'bg-grey-darken-4 text-white' : 'bg-grey-lighten-3'">
      <v-container fluid class="py-6 px-4 px-sm-6">
        <v-row justify="center">
          <v-col cols="12" sm="10" md="8">
            <v-sheet
              :class="currentTheme === 'dark' ? 'bg-grey-darken-3 text-white' : 'bg-white'"
              class="pa-4 pa-sm-6 text-center"
              elevation="2"
              rounded="lg"
            >
              <!-- Title -->
              <h1 class="text-h5 text-md-h4 font-weight-bold mb-4">About</h1>
              <!-- Future content goes here -->
            </v-sheet>
          </v-col>
        </v-row>
      </v-container>
    </v-main>
  </v-app>
</template>
