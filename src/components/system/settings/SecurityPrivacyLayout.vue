<script setup>
import { ref } from 'vue';

const securityGuidelines = ref([
  {
    title: 'Protecting Your Data',
    description:
      'We ensure your personal information is encrypted and stored securely, protecting it from unauthorized access in compliance with the Philippines’ Data Privacy Act.',
    icon: 'mdi-shield-check',
  },
  {
    title: 'Secure and Ethical Use of the Platform',
    description:
      'Posting inappropriate content, including nudity, violence, or malicious information, is strictly prohibited. Violators will face penalties, including restrictions or permanent bans.',
    icon: 'mdi-account-lock',
  },
  {
    title: 'Moderation and Reporting',
    description:
      'Our active moderators monitor the site to maintain a safe and respectful environment. Users can report content or activities that violate our guidelines.',
    icon: 'mdi-eye-off',
  },
  {
    title: 'Transparent Data Practices',
    description:
      'We are committed to transparency about how your data is collected, used, and shared. You have full control over your personal information and can request updates or deletion at any time.',
    icon: 'mdi-lock-alert',
  },
  {
    title: 'Safe Online Interactions',
    description:
      'To protect everyone, avoid sharing sensitive personal details (like passwords or financial information) in public spaces.',
    icon: 'mdi-wifi-lock',
  },
  {
    title: 'Always Use Secure Connections',
    description:
      'Our platform runs on HTTPS to ensure all communication between you and the website is encrypted and safe from third-party attacks.',
    icon: 'mdi-wifi-lock',
  },
  {
    title: 'Strong Content Filters',
    description:
      'We use advanced filters to block harmful or inappropriate content before it is visible to others.',
    icon: 'mdi-filter-variant',
  },
  {
    title: 'User Accountability',
    description:
      'All activities are logged securely. Users found violating our terms may be restricted, and repeat offenders may face permanent bans.',
    icon: 'mdi-account-alert',
  },
  {
    title: 'Session Security',
    description:
      'Your account is protected with secure logins and automatic logout after inactivity to prevent unauthorized access.',
    icon: 'mdi-logout-variant',
  },
  {
    title: 'Your Privacy Matters',
    description:
      'We respect your privacy and follow strict protocols to ensure your data is protected. Detailed policies can be found in our Privacy Policy document.',
    icon: 'mdi-shield-account',
  },
]);
</script>
<template>
  <v-container class="py-8">
    <v-row class="d-flex justify-center" dense>
      <v-col
        v-for="(guideline, index) in securityGuidelines"
        :key="index"
        cols="12"
        md="4"
        lg="3"
        class="mb-4"
      >
        <v-card class="elevation-2 rounded-lg text-center pa-4 card-fixed-size">
          <!-- Use mdi icon directly in the v-icon component -->
          <v-icon :icon="guideline.icon" size="48" color="#4caf50" class="mb-3" />
          <div class="font-weight-bold text-h6">
            {{ guideline.title }}
          </div>
          <p class="text-body-2 text-secondary mt-2">{{ guideline.description }}</p>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Matemasie&family=Varela+Round&display=swap');

*{
  font-family: 'Varela Round', sans-serif;
  font-weight: 400;
  font-style: normal;
}
.card-fixed-size {
  height: 300px; /* Adjust height to ensure uniformity */
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  align-items: center;
}

.v-card:hover {
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.1);
  transform: translateY(-4px);
  transition: all 0.3s ease-in-out;
}
</style>
