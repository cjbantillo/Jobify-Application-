<script setup>
import { ref, onMounted } from "vue";
import { supabase } from '@/utils/supabase'
import { useAuthUserStore } from '@/stores/authUser'
import router from "@/router";

const resumeFile = ref(null)
const fileName = ref('')
const fileInput = ref(null)
const resumeUrl = ref('')
const loading = ref(true)
const isDragging = ref(false) // Track dragging state

const authStore = useAuthUserStore()

const user_name = `${authStore.userData.first_name || ''} ${authStore.userData.last_name || ''} Resume`

// Supabase bucket and file details
const bucketName = 'resumes' // Replace with your Supabase bucket name
const filePath = `${user_name}.pdf` // Use user name and unique UUID

// Fetch existing resume by ID (filePath)
// Fetch existing resume URL
const fetchResume = async () => {
  try {
    loading.value = true

    // Generate the file path (replace with a user-specific file path logic)
    // Example: const filePath = `${userId}/resume.pdf`;
    const { data, error } = supabase.storage
      .from(bucketName)
      .getPublicUrl(filePath) // Correct use of getPublicUrl

    if (error) {
      throw new Error(error.message)
    }

    if (data.publicUrl) {
      resumeUrl.value = data.publicUrl // Set resume URL to the fetched public URL
    } else {
      resumeUrl.value = '' // File not found
    }
  } catch (error) {
    console.error('Error fetching resume:', error.message)
    resumeUrl.value = '' // Clear URL on error
  } finally {
    loading.value = false
  }
}

// Handle file selection
const handleFileUpload = () => {
  if (fileInput.value?.files[0]) {
    processFile(fileInput.value.files[0])
  }
}

// Handle file drop
const handleDrop = event => {
  event.preventDefault()
  isDragging.value = false
  if (event.dataTransfer.files[0]) {
    processFile(event.dataTransfer.files[0])
  }
}

// Process the selected or dropped file
const processFile = file => {
  resumeFile.value = file
  fileName.value = file.name
}

// Upload file to Supabase
const uploadResume = async () => {
  if (!resumeFile.value) return

  try {
    loading.value = true
    const { error } = await supabase.storage
      .from('resumes')
      .upload(filePath, resumeFile.value, {
        cacheControl: '3600',
        upsert: true,
        contentType: 'application/pdf',
      })

    if (error) throw error

    router.push('/resume');
    await fetchResume() // Refresh to show the uploaded file
    alert('Resume uploaded successfully!')
  } catch (error) {
    console.error('Error uploading resume:', error.message)
    alert('Failed to upload resume. Please try again.')
  } finally {
    loading.value = false
  }
}

// Delete the resume
const deleteResume = async () => {
  try {
    const { error } = await supabase.storage.from(bucketName).remove([filePath])

    if (error) throw error

    resumeFile.value = null
    fileName.value = ''
    resumeUrl.value = ''
    alert('Resume deleted successfully!')
  } catch (error) {
    console.error('Error deleting resume:', error.message)
    alert('Failed to delete resume. Please try again.')
  }
}

// Fetch resume on component mount
onMounted(() => {
  fetchResume()
})
</script>
<template>
  <v-container>
    <v-row justify="center">
      <v-col cols="12" md="10" xl="8">
        <v-card class="pa-8 hover-card move-down">
          <v-card-title class="title">Resume</v-card-title>
          <v-card-subtitle class="subtitle">
            Manage your resume easily and keep track of your applications.
          </v-card-subtitle>
          <v-card-text>
            <div
              class="upload-section"
              @drop="handleDrop"
              @dragover.prevent="isDragging = true"
              @dragleave="isDragging = false"
              :class="{ dragging: isDragging }"
            >
              <h3>Upload Your Resume</h3>
              <input
                type="file"
                accept=".pdf,.doc,.docx"
                style="display: none"
                ref="fileInput"
                @change="handleFileUpload"
              />
              <v-btn color="#4caf50" outlined @click="$refs.fileInput.click()"
                >Choose a file</v-btn
              >
              <div class="drop-area">
                <p>Or drag and drop a file here</p>
              </div>
              <div v-if="fileName" class="file-name">
                Selected file: {{ fileName }}
              </div>
              <div class="upload-actions" v-if="resumeFile">
                <v-btn color="#4caf50" outlined @click="uploadResume"
                  >Upload</v-btn
                >
                <v-btn color="error" outlined @click="deleteResume"
                  >Delete</v-btn
                >
              </div>
            </div>

            <v-divider class="my-4"></v-divider>

            <div class="info-section">
              <h3>Welcome to Your Resume</h3>
              <p>
                Here, you can upload your resume, view and delete it as needed.
                Keep track of your professional journey efficiently.
              </p>
              <div v-if="resumeUrl" class="resume-preview">
                <h3>Your Uploaded Resume:</h3>
                <iframe
                  :src="resumeUrl"
                  width="100%"
                  height="500px"
                  style="border: none"
                ></iframe>
              </div>
              <div v-else>
                <p>No resume uploaded. Please upload one to view it here.</p>
              </div>
            </div>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<style scoped src="./ResumeDashboardsStyle.css">

</style>
