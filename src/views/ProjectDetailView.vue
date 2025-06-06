<template>
  <div class="project-detail-page bg-[#0f172a] min-h-screen p-8">
    <!-- Navigation vers l'accueil -->
    <div class="max-w-7xl mx-auto mb-8 flex justify-center pt-48 gap-12">
      <router-link to="/project" class="text-maingreen hover-underline-animation">
        Retour à l'accueil
      </router-link>
      <router-link to="/project" class="text-maingreen hover-underline-animation">
        Retour aux projets
      </router-link>
    </div>

    <!-- Contenu du projet -->
    <div v-if="currentProject" class="max-w-4xl mx-auto pt-16">
      <div class="mb-8">
        <h1 class="text-4xl font-bold text-white mb-4 gradient-text">
          {{ currentProject.title }}
        </h1>
        <div class="flex flex-wrap gap-2 mb-6">
          <span
            v-for="tech in currentProject.technologies"
            :key="tech"
            class="px-3 py-1 bg-[#1e293b] text-maingreen rounded"
          >
            {{ tech }}
          </span>
        </div>
      </div>

      <div class="grid grid-cols-1 md:grid-cols-2 gap-8 mb-12">
        <div class="space-y-6">
          <img
            :src="currentProject.imageUrl"
            :alt="currentProject.title"
            class="w-full rounded-lg shadow-lg"
          />
<!-- Galerie d'images supplémentaires -->
<div v-if="currentProject.imageUrls" class="grid grid-cols-2 md:grid-cols-3 gap-4 mt-6">
  <img
    v-for="(img, index) in currentProject.imageUrls"
    :key="index"
    :src="img"
    class="rounded-lg cursor-pointer hover:opacity-80 transition-all"
    @click="openLightbox(img)"
  />
</div>


          <div class="flex gap-4">
            <a
              :href="currentProject.demoLink"
              target="_blank"
              class="flex-1 px-6 py-2 bg-maingreen text-white rounded text-center hover:bg-opacity-90 transition-all"
            >
              Voir le projet
            </a>
            <a
              :href="currentProject.githubLink"
              target="_blank"
              class="flex-1 px-6 py-2 border border-maingreen text-maingreen rounded text-center hover:bg-maingreen hover:text-white transition-all"
            >
              Code source
            </a>
          </div>
        </div>

        <div class="space-y-6">
          <div>
            <h2 class="text-2xl font-bold text-white mb-3">Description</h2>
            <p class="text-gray-300 leading-relaxed">
              {{ currentProject.longDescription }}
            </p>
          </div>

          <div>
            <h2 class="text-2xl font-bold text-white mb-3">Fonctionnalités</h2>
            <ul class="list-disc list-inside text-gray-300 space-y-2">
              <li v-for="feature in currentProject.features" :key="feature">
                {{ feature }}
              </li>
            </ul>
          </div>
        </div>
      </div>

      <!-- Navigation entre projets -->
      <div
        class="flex justify-between items-center mt-16 pt-8 border-t border-gray-700"
      >
        <button
          @click="navigateProject('prev')"
          class="text-maingreen hover:text-white transition-colors"
          :class="{ 'opacity-50 cursor-not-allowed': !hasPrevProject }"
          :disabled="!hasPrevProject"
        >
          <div class="flex items-center gap-2">
            <Icon icon="mdi:chevron-left" width="32" height="32" />
            <span class="text-sm">Projet précédent</span>
          </div>
        </button>

        <button
          @click="navigateProject('next')"
          class="text-maingreen hover:text-white transition-colors"
          :class="{ 'opacity-50 cursor-not-allowed': !hasNextProject }"
          :disabled="!hasNextProject"
        >
          <div class="flex items-center gap-2">
            <span class="text-sm">Projet suivant</span>
            <Icon icon="mdi:chevron-right" width="32" height="32" />
          </div>
        </button>
      </div>
    </div>

    <!-- Message d'erreur si projet non trouvé -->
    <div v-else class="text-white text-center">Projet non trouvé</div>
  </div>
  <!-- Lightbox -->
<div
  v-if="lightboxImage"
  class="fixed inset-0 bg-black bg-opacity-80 flex items-center justify-center z-50"
  @click.self="closeLightbox"
>
  <img :src="lightboxImage" class="max-w-full max-h-full rounded-lg shadow-xl" />
  <button
    @click="closeLightbox"
    class="absolute top-6 right-6 text-white text-3xl"
  >
    &times;
  </button>
</div>

</template>

<script>
import { ref, computed } from "vue";
import { useRouter, useRoute } from "vue-router";
import { useProjectStore } from "@/stores/projectStore";
import { Icon } from "@iconify/vue";

export default {
  name: "ProjectDetailView",
  components: {
    Icon,
  },
  setup() {
    const router = useRouter();
    const route = useRoute();
    const store = useProjectStore();
    const currentProjectId = ref(parseInt(route.params.id));
    const lightboxImage = ref(null);

    // Fonction pour ouvrir la lightbox
    const openLightbox = (image) => {
      lightboxImage.value = image;
    };

    // Fonction pour fermer la lightbox
    const closeLightbox = () => {
      lightboxImage.value = null;
    };

    // Récupérer le projet actuel
    const currentProject = computed(() =>
      store.getProjectById(currentProjectId.value)
    );

    // Récupérer tous les projets pour la navigation
    const allProjects = store.getAllProjects;

    // Calculer les indices pour la navigation
    const currentIndex = computed(() =>
      allProjects.findIndex((p) => p.id === currentProjectId.value)
    );

    const hasPrevProject = computed(() => currentIndex.value > 0);
    const hasNextProject = computed(
      () => currentIndex.value < allProjects.length - 1
    );

    // Fonction de navigation entre les projets
    const navigateProject = (direction) => {
      let newIndex;
      if (direction === "next" && hasNextProject.value) {
        newIndex = currentIndex.value + 1;
      } else if (direction === "prev" && hasPrevProject.value) {
        newIndex = currentIndex.value - 1;
      } else {
        return;
      }

      const newProjectId = allProjects[newIndex].id;
      router.push({ name: "project-detail", params: { id: newProjectId } });
      currentProjectId.value = newProjectId;
    };

    return {
      currentProject,
      hasPrevProject,
      hasNextProject,
      navigateProject,
      openLightbox,
      closeLightbox,
      lightboxImage,
    };
  },
};
</script>

<style scoped>
 .gradient-text {
  background: linear-gradient(300deg, #0ecff6, #f2f2f2, #2df3ec, #22f898, #13cef8);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  display: inline-block;
  background-size: 200% auto;
  animation: gradientMove 3s linear infinite;
}

@keyframes gradientMove {
  0% {
    background-position: 0% 50%;
  }
  50% {
    background-position: 100% 50%;
  }
  100% {
    background-position: 0% 50%;
  }
}

.hover-underline-animation {
  position: relative;
  text-decoration: none;
}

.hover-underline-animation::after {
  content: "";
  position: absolute;
  height: 2px;
  width: 100%;
  bottom: 0;
  left: 0;
  background-color: currentColor;
  transform: scaleX(0);
  transform-origin: left;
  transition: transform 0.3s ease-out;
}

.hover-underline-animation:hover::after {
  transform: scaleX(1);
}
</style>
