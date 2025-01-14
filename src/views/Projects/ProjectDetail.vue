<template>
  <v-container fluid>
    <project-bar></project-bar>
    <v-tabs vertical background-color="#f7f7f7" class="project-detail__wrapper"
        v-if="allProjects && allProjects.results.length">
      <div class="project-detail__top-tab">{{ $t('projects.browseProjects') }}</div>
      <v-tab
        class="text-left"
        height="130px"
        :to="'/project/' + project.id"
        v-for="project in allProjects.results"
        :key="project.id"
        v-model="tab"
      >
        <v-row no-gutters>
          <v-col cols="12" class="mb-2">
            <h4 class="mb-4">{{ project.title }}</h4>
            <v-divider></v-divider>
          </v-col>
          <v-col cols="6">
            <h4>{{ project.decks.length }}</h4>
            <small>{{ $t('projects.numberDecks') }}</small>
          </v-col>
          <v-col cols="6" class="text-right">
            <h4>{{ verboseDate(project.currentCommitDateTime) }}</h4>
            <small>{{ $t('projects.lastUpdate') }}</small>
          </v-col>
        </v-row>
      </v-tab>
      <v-tabs-items v-model="tab">
        <v-tab-item v-if="project">
          <v-container v-if="!$route.query.edit" class="px-8 py-5">
            <v-row>
              <v-col cols="6">
                <h2 class="mb-1">{{ project.title }}</h2>
                <small class="d-block">{{ project.description }}</small>
                <small class="mt-3">
                  <b class="mr-3">{{ $t('projects.lastUpdate') }}</b>
                  {{ verboseDate(project.currentCommitDateTime) }}
                </small>
              </v-col>
              <v-col cols="3">
                <h4 class="mb-1">{{ verboseDate(project.created) }}</h4>
                <small class="d-block">{{ $t('projects.dateAdded') }}</small>
              </v-col>
              <v-col cols="3">
                <v-btn
                  class="float-right"
                  outlined
                  width="144"
                  @click="setEdit"
                  :ripple="false"
                >
                  <v-icon size="24" class="mr-2">
                    $vuetify.icons.edit
                  </v-icon>{{ $t('projects.editProject') }}</v-btn>

                <v-btn
                  class="float-right mt-5 "
                  outlined
                  width="144"
                  :ripple="false"
                  @click="showDeleteDialog = true;"
                >
                  <v-icon size="24" class="mr-2">
                    $vuetify.icons.delete
                  </v-icon>{{ $t('projects.deleteProject') }}</v-btn>
              </v-col>
            </v-row>
            <v-row>
              <v-col cols="6">
              </v-col>
              <v-col cols="3" v-if="project.creator">
                <h4>{{ project.creator.firstName }} {{ project.creator.lastName }}</h4>
                <small class="d-block">{{ $t('projects.createdBy') }}</small>
              </v-col>
            </v-row>
            <v-divider></v-divider>
            <v-tabs v-model="innerTab" height="67px" class="project-detail__tabs" slider-size="3">
              <v-tab :ripple="false" :to="{name: 'project.detail.decks'}">
                {{ $tc('deck.Deck', 2) }}
                <span class="tab-count-badge">
                  {{ project.decks.length.toString().padStart(2, '0') }}
                </span>
              </v-tab>
              <v-tab v-if="project.sops" :ripple="false"
                  :to="{name: 'project.detail.sops'}">
                {{ $t('projects.sops') }}
                <span class="tab-count-badge">
                  {{ project.sops.length.toString().padStart(2, '0') }}
                </span>
              </v-tab>
              <v-tab v-if="project.members" :ripple="false" :to="{name: 'project.detail.members'}">
                {{ $t('projects.members') }}
                <span class="tab-count-badge">
                  {{ project.members.length.toString().padStart(2, '0') }}
                </span>
              </v-tab>
              <v-tab v-if="project.clusterSettings" :ripple="false"
                  :to="{name: 'project.detail.clusterSettings'}">
                {{ $t('projects.clusterSettings') }}
              </v-tab>
            </v-tabs>
            <div class="py-8">
              <router-view :project="project" @update="updateProject"></router-view>
            </div>
          </v-container>
          <v-container v-else>
            <div class="px-3">
              <span class="text--disabled" v-if="project">{{ project.title }} ></span>
              <h2 class="text--semi-bold">{{ $t('projects.editProject')}}</h2>
              <v-divider></v-divider>
            </div>
            <project-form
              :edit-mode="true"
              :project="project"
              @sops-created="handleSopsCreated"
            ></project-form>
          </v-container>
        </v-tab-item>
      </v-tabs-items>
    </v-tabs>
    <v-navigation-drawer
      temporary
      right
      absolute
      width="650"
      light
      class="no-bg-drawer"
      v-model="memberDrawer"
    >
      <add-team-member
        v-if="project"
        :overlay="true"
        :project="project"
        v-on:done="memberDrawer = false"
      ></add-team-member>
    </v-navigation-drawer>
    <delete-project :show="showDeleteDialog" :project="project"
        @hide="showDeleteDialog = false;"
        @deleted="$router.push({name: 'overview'});"></delete-project>
  </v-container>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';
import ProjectBar from '@/components/Projects/ProjectBar.vue';
import ProjectForm from '@/views/Projects/ProjectForm.vue';
import DeleteProject from '@/components/Projects/DeleteProject.vue';
import AddTeamMember from '@/views/Projects/AddTeamMember.vue';
import EditDeck from '@/components/Projects/EditDeck.vue';
import UnikubeAvatar from '@/components/general/Avatar.vue';
import {
  ProjectDetailQuery,
  ProjectDetailOtherProjectsQuery,
  TDeckNode,
  TProjectNode,
} from '@/generated/graphql';

@Component({
  components: {
    ProjectBar,
    ProjectForm,
    AddTeamMember,
    EditDeck,
    UnikubeAvatar,
    DeleteProject,
  },
  apollo: {
    project: {
      query: ProjectDetailQuery,
      variables() {
        return {
          id: this.projectSlug,
        };
      },
    },
    allProjects: {
      query: ProjectDetailOtherProjectsQuery,
    },
  },
  beforeRouteUpdate(to, from, next) {
    this.$apollo.queries.allProjects.refetch();
    this.$apollo.queries.project.refresh();
    next();
  },
})
export default class ProjectDetail extends Vue {
  get projectSlug(): string {
    return this.$route.params.slug;
  }

  tab = 0;

  showDeleteDialog = false;

  innerTab = 0;

  deckEdit = false;

  deckToBeEdited: TDeckNode | undefined;

  memberDrawer = false;

  project!: TProjectNode

  verboseDate(date: Date): string {
    if (date) {
      return this.$d(new Date(date), 'short');
    }
    return '';
  }

  setEdit(): void {
    this.$router.push({ query: { edit: 'true' } });
  }

  setDeckEdit(pkg: TDeckNode): void {
    this.deckToBeEdited = pkg;
    this.deckEdit = true;
  }

  handleSopsCreated(): void {
    this.updateProject();
  }

  updateProject(): void {
    this.$apollo.queries.project.refetch();
  }
}
</script>

<style lang="scss" scoped>
.v-tabs--vertical > .v-tabs-bar .v-tab {
  height: 130px !important;
}

.project-detail {
  &__wrapper {
    box-shadow: 0 2px 40px 0 rgba(183, 183, 183, 0.15);
    border-radius: 8px;
  }

  &__top-tab {
    background-color: #9eaed7;
    color: white;
    padding: 14px 23px;
    border-top-left-radius: 8px;
  }
}

.project-detail__tabs {
  .v-tab {
    padding: 0 36px;
  }
}

.deployment-badge {
  font-weight: 500;
  background-color: #f3f4f9;
  color: #9eaed7;
  white-space: nowrap;
  flex: 0 0 auto;
  margin-bottom: 10px;
}

.tab-count-badge {
  background-color: $unikube-green;
  color: white;
  padding: 4px 8px;
  font-size: 12px;
  margin-left: 8px;
  border-radius: 4px;
}

.theme--light.v-data-table > .v-data-table__wrapper > table > thead > tr > th {
  color: white;
}
</style>
