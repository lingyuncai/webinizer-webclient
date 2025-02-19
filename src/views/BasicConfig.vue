<!--
  *  Copyright (C) 2023 Intel Corporation. All rights reserved.
  *  Licensed under the Apache License 2.0. See LICENSE in the project root for license information.
  *  SPDX-License-Identifier: Apache-2.0
  -->

<template>
  <section class="banner flex-center text-center b1">
    <div>
      <h1>{{ getProjectName(route.query.root as string, config) }} Basic Configuration</h1>
      <h2>Customize your basic configurations</h2>
    </div>
  </section>

  <div :class="['mx-auto', 'w-75', 'mt-8', 'basic', 'text-left', showDependentArea ? 'g1x2' : '']">
    <div class="px-4 py-0 bl mb-6 mr-6">
      <v-card variant="flat" :disabled="projAddStatus === ProjectAddStatus.processing">
        <v-card-item class="text-left mb-8"> <v-card-title>Basic</v-card-title> </v-card-item>

        <v-card-text v-if="!root" class="mt-n8">
          <v-tabs
            v-model="tabValue"
            color="blue"
            @update:model-value="
              (selectedType = $event),
                selectedType === AddProjectType.registry ? (projIsLibrary = false) : ''
            "
            ><v-tab :value="AddProjectType.upload">
              <span v-if="!showDependentArea" class="hidden-md-and-down"
                >upload project locally</span
              >
              <v-icon title="Upload project locally" end> mdi-file-upload </v-icon> </v-tab
            ><v-tab :value="AddProjectType.clone">
              <span v-if="!showDependentArea" class="hidden-md-and-down">clone from github</span>
              <v-icon title="Clone from github" end> mdi-github </v-icon></v-tab
            >

            <template v-if="!settings?.registry">
              <span
                :value="AddProjectType.registry"
                title="Add a valid registry address in the Settings page to enable adding a project from registry.">
                <v-tab :value="AddProjectType.registry" disabled>
                  <span v-if="!showDependentArea" class="hidden-md-and-down"
                    >add from registry</span
                  >
                  <v-icon title="Add from registry" end> mdi-cloud-download </v-icon></v-tab
                >
              </span>
            </template>

            <template v-else>
              <v-tab :value="AddProjectType.registry">
                <span v-if="!showDependentArea" class="hidden-md-and-down">add from registry</span>
                <v-icon title="Add from registry" end> mdi-cloud-download </v-icon></v-tab
              ></template
            >
          </v-tabs>

          <v-window v-model="tabValue" class="mt-4">
            <v-window-item :value="AddProjectType.upload">
              <v-form ref="formUpload" class="mt-8" @submit.prevent>
                <div class="d-flex">
                  <div class="pr-4 mt-n4 align-self-baseline flex-grow-0 flex-shrink-0">
                    <Icons
                      :icons="icons"
                      :selected-icon="selectedIcon"
                      :upload-icon="uploadIcon"
                      @upload="iconChangeHandler"
                      @change="selectIconHandler"
                      @clear="uploadIcon = []"
                      @get-icons="getAvailableIcons"></Icons>
                  </div>

                  <div class="flex-grow-1 flex-shrink-0">
                    <div class="g1x2r" style="grid-column-gap: 1rem">
                      <EditTextField
                        no-need-save-instantly
                        :label="'Project name'"
                        :value="projName"
                        :rules="formFieldRulesObject.fieldNameRules"
                        @change="
                          (name) => {
                            projName = name;
                          }
                        "></EditTextField>

                      <EditTextField
                        no-need-save-instantly
                        :label="'Project version'"
                        :value="projVersion"
                        :rules="formFieldRulesObject.fieldVersionRules"
                        @change="
                          (version) => {
                            projVersion = version;
                          }
                        "></EditTextField>
                    </div>
                  </div>
                </div>

                <div class="mt-8">
                  <EditTextAreaVue
                    no-need-save-instantly
                    :rows="2"
                    :label="'Project description'"
                    :value="projDesc"
                    @change="
                      (desc) => {
                        projDesc = desc;
                      }
                    "></EditTextAreaVue>
                </div>

                <v-file-input
                  :show-size="1024"
                  :rules="formFieldRulesObject.fileInputRules"
                  class="mt-2"
                  color="blue"
                  label="Select the project ZIP file from local"
                  validate-on="input"
                  accept=".zip"
                  variant="outlined"
                  @change="fileChangeHandler($event)">
                  <template #selection="{ fileNames }">
                    <template v-for="fileName in fileNames" :key="fileName">
                      <v-chip size="small" label color="blue" class="me-1">
                        {{ fileName }}
                      </v-chip>
                    </template>
                  </template></v-file-input
                >
                <v-progress-linear
                  color="blue"
                  :model-value="refUploadProgress"></v-progress-linear>
                <v-checkbox
                  class="mt-4 mb-n5"
                  color="blue"
                  label="Check the box if the project is a library."
                  :model-value="projIsLibrary"
                  @change="projIsLibrary = !projIsLibrary"></v-checkbox>
              </v-form>
            </v-window-item>

            <v-window-item :value="AddProjectType.clone">
              <v-form ref="formClone" class="mt-8" @submit.prevent>
                <div class="d-flex">
                  <div class="pr-4 mt-n4 align-self-baseline flex-grow-0 flex-shrink-0">
                    <Icons
                      :icons="icons"
                      :selected-icon="selectedIcon"
                      :upload-icon="uploadIcon"
                      @upload="iconChangeHandler"
                      @change="selectIconHandler"
                      @clear="uploadIcon = []"
                      @get-icons="getAvailableIcons"></Icons>
                  </div>
                  <div class="flex-grow-1 flex-shrink-0">
                    <div class="g1x2r" style="grid-column-gap: 1rem">
                      <EditTextField
                        no-need-save-instantly
                        :label="'Project name'"
                        :value="projName"
                        :rules="formFieldRulesObject.fieldNameRules"
                        @change="
                          (name) => {
                            projName = name;
                          }
                        "></EditTextField>

                      <EditTextField
                        no-need-save-instantly
                        :label="'Project version'"
                        :value="projVersion"
                        :rules="formFieldRulesObject.fieldVersionRules"
                        @change="
                          (version) => {
                            projVersion = version;
                          }
                        "></EditTextField>
                    </div>
                  </div>
                </div>

                <div class="mt-8">
                  <EditTextAreaVue
                    no-need-save-instantly
                    :rows="2"
                    :label="'Project description'"
                    :value="projDesc"
                    @change="
                      (desc) => {
                        projDesc = desc;
                      }
                    "></EditTextAreaVue>
                </div>

                <div class="mt-8">
                  <EditTextAreaVue
                    no-need-save-instantly
                    :rows="2"
                    :rules="formFieldRulesObject.fieldRequiredRules"
                    :label="'Path of remote git repository'"
                    @change="
                      (path) => {
                        repoPath = path;
                      }
                    "></EditTextAreaVue>
                </div>

                <v-checkbox
                  class="mb-n4"
                  color="blue"
                  label="Check the box if the project is a library."
                  :model-value="projIsLibrary"
                  @change="projIsLibrary = !projIsLibrary"></v-checkbox>
              </v-form>
            </v-window-item>

            <v-window-item :value="AddProjectType.registry">
              <v-form ref="formRegistry" class="mt-2" @submit.prevent>
                <v-combobox
                  v-model="projName"
                  label="Package name"
                  variant="outlined"
                  clearable
                  open-on-clear
                  :rules="formFieldRulesObject.fieldNameRules"
                  :items="registryPkgSearchList"
                  @input="queryRegistryWithDebounce($event.srcElement.value)"
                  @keyup.enter="confirmPackageHandler($event.srcElement.value)"
                  @blur="
                    {
                      confirmPackageHandler($event.srcElement.value),
                        (searchResultsFromRegistry = []);
                    }
                  "
                  @update:menu="confirmPackageHandler(projName as string)"></v-combobox>
                <div class="mt-8">
                  <EditTextField
                    no-need-save-instantly
                    :rules="formFieldRulesObject.fieldVersionRules"
                    :label="'Project version'"
                    :value="projVersion"
                    @change="
                      (version) => {
                        version ? (projVersion = version) : '';
                      }
                    "></EditTextField>
                </div>
              </v-form>
            </v-window-item>
          </v-window>
        </v-card-text>

        <v-card-text v-if="!root" class="text-center">
          <v-btn
            color="info"
            :loading="projAddStatus === ProjectAddStatus.processing ? true : false"
            @click="submitProject()"
            >CREATE</v-btn
          >
        </v-card-text>

        <v-card-text v-if="root">
          <div class="d-flex">
            <div class="pr-4 mt-n4 align-self-baseline flex-grow-0 flex-shrink-0">
              <Icons
                show-action
                :exist-upload-icon="existUploadIcon"
                :icons="icons"
                :selected-icon="selectedIcon"
                :upload-icon="uploadIcon"
                @upload="iconChangeHandler"
                @change="selectIconHandler"
                @clear="uploadIcon = []"
                @get-icons="getAvailableIcons"
                @upload-immediately="uploadProjectIcon"
                @remove-icon="removeIcon"></Icons>
            </div>
            <div class="flex-grow-1 flex-shrink-0">
              <div class="g1x2r" style="grid-column-gap: 1rem">
                <EditTextField
                  read-only
                  :label="'Project name'"
                  :value="config?.name"
                  @change="saveProjectName"></EditTextField>

                <EditTextField
                  :label="'Project version'"
                  :value="config?.version"
                  :rules="formFieldRulesObject.fieldVersionRules"
                  @change="saveProjectVersion"></EditTextField>
              </div>
            </div>
          </div>
        </v-card-text>

        <v-card-text v-if="root">
          <EditTextAreaVue
            :rows="2"
            :label="'Project description'"
            :value="config?.desc"
            @change="saveProjectDesc"></EditTextAreaVue
        ></v-card-text>

        <v-card-text v-if="root">
          <EditTextAreaVue
            read-only
            :rows="2"
            :label="'Path to native project'"
            :value="root"></EditTextAreaVue>
        </v-card-text>

        <v-card-text v-if="root">
          <v-checkbox
            class="mt-n4"
            color="blue"
            :model-value="config?.isLibrary === undefined ? false : config?.isLibrary"
            @change="saveProjectIsLib()">
            <template #label
              ><text
                >Check the box if the project is a
                <a
                  class="modulelink"
                  title="click to edit configuration"
                  @click="
                    router.push({
                      path: `/config`,
                      query: { configParameterType: ConfigParameterTypes.nativeLibrary },
                    })
                  "
                  @click.stop
                  >library</a
                >&nbsp;&nbsp;&nbsp;&nbsp;
              </text></template
            ></v-checkbox
          >
        </v-card-text>

        <v-card-text class="text-right">
          <v-btn
            v-if="showDependentArea"
            append-icon="mdi-chevron-left"
            variant="text"
            color="#42a5f5"
            @click="showDependentArea = !showDependentArea">
            <span title="Click to add dependencies of your project"> Project Dependencies </span>
          </v-btn>

          <v-btn
            v-else
            append-icon="mdi-chevron-right"
            variant="text"
            color="#42a5f5"
            @click="showDependentArea = !showDependentArea">
            <span title="Click to add dependencies of your project"> Project Dependencies </span>
          </v-btn>
        </v-card-text>
      </v-card>
    </div>

    <div v-if="showDependentArea" class="pa-4 py-0 bl mb-6">
      <v-card variant="flat" :disabled="projAddStatus === ProjectAddStatus.processing">
        <v-card-item>
          <v-card-title
            >Dependencies
            <v-btn
              color="info"
              size="small"
              variant="text"
              @click="
                router.push({
                  path: `/config`,
                  query: { configParameterType: ConfigParameterTypes.dependency },
                })
              ">
              <template #prepend> <v-icon icon="mdi-pencil" class="mr-n2"></v-icon> </template
              >Edit</v-btn
            ></v-card-title
          ></v-card-item
        >

        <v-list lines="two">
          <v-list-item v-for="(item, idx) in dependentProjectNameToDisplay" :key="idx" class="mb-4">
            <template #title>
              <span class="text-subtitle-1 font-weight-medium"> {{ item.name }}</span>
            </template>

            <template #subtitle>
              <span :title="item.desc">{{ item.desc }}</span>
            </template>

            <template #append>
              <v-icon class="delete-icon" icon="mdi-close-thick" @click="showAlert(idx)"></v-icon>
            </template>
          </v-list-item>
        </v-list>

        <v-card class="mx-auto">
          <v-card-text v-if="showDependencySelect" class="mt-n2 mb-n8">
            <v-combobox
              v-model="dependentProjectNameSelected"
              multiple
              clearable
              single-line
              label="Select dependency"
              style="word-break: break-all"
              density="compact"
              :items="dependencySelectOptions"
              @input="queryRegistryWithDebounce($event.srcElement.value)"
              @click:clear="searchResultsFromRegistry = []"
              @keyup.enter="triggerAddDependencies"
              @blur="
                {
                  if (!root) triggerAddDependencies();
                  searchResultsFromRegistry = [];
                }
              "></v-combobox>
          </v-card-text>

          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn
              v-if="!showDependencySelect"
              variant="outlined"
              color="blue"
              :loading="projDepUpdateStatus === ProjectDepUpdateStatus.processing ? true : false"
              @click="showDependencySelect = true">
              ADD DEPENDENCIES
            </v-btn>

            <v-btn
              v-else-if="root"
              variant="outlined"
              color="blue"
              title="click to confirm"
              :loading="projDepUpdateStatus === ProjectDepUpdateStatus.processing ? true : false"
              @click="triggerAddDependencies">
              FINISH </v-btn
            ><v-spacer></v-spacer>
          </v-card-actions>
        </v-card>
      </v-card>
    </div>
  </div>

  <div class="mx-auto w-75">
    <v-card flat>
      <v-card-actions>
        <v-spacer></v-spacer>
        <v-btn
          append-icon="mdi-arrow-right"
          color="info"
          @click="router.push({ path: '/buildSteps' })">
          BUILD STEPS</v-btn
        >
      </v-card-actions>
    </v-card>
  </div>

  <Transition name="slide-fade">
    <Alert
      v-if="alert"
      content="Are you sure to delete?"
      @confirm="deleteDependencies()"
      @cancel="alert = false"></Alert
  ></Transition>
</template>

<script setup lang="ts">
/*
  eslint-disable
    @typescript-eslint/no-unsafe-call,
    no-unsafe-optional-chaining,
    @typescript-eslint/no-unsafe-assignment
 */

import EditTextAreaVue from "../components/config/EditTextArea.vue";
import EditTextField from "../components/config/EditTextField.vue";
import Alert from "../components/Alert.vue";
import Icons from "../components/Icons.vue";

import { useStore } from "../store";
import { useRoute, useRouter } from "vue-router";
import { computed, onMounted, ref, watch } from "vue";
import { getProjectName } from "../common/utility/utility";
import { cloneDeep } from "lodash";
import { log } from "../helper";
import { convertDate } from "../common/utility/utility";
import {
  API_SERVER,
  ProjectAddStatus,
  ProjectDepUpdateStatus,
  PackageSearchResult,
  search as searchRegistry,
  ConfigParameterTypes,
  ProjectIcon,
} from "../webinizer";
import { useToast } from "vue-toastification";
import lodash from "lodash";

//NOTE - Here we should keep the order of the config.deps
//       so that the dependency project added later will
//       append to the list, otherwise the list order will
//       follow the order in projects projectPool
type DependentProjInfo = {
  name: string;
  desc: string;
  path?: string;
};

enum AddProjectType {
  upload = "upload",
  clone = "clone",
  registry = "registry",
}

const toast = useToast();
const route = useRoute();
const store = useStore();
const router = useRouter();
let deleteIdx = -1;
const projectChunkSize = 512 * 1024;
const localPrefixStr = "[ Local ] - ";
const remotePrefixStr = "[ Remote ] - ";
const formFieldRulesObject = {
  fieldRequiredRules: [
    (value: string) => {
      if (value && value.trim()?.length > 0) return true;
      return "Field is required.";
    },
  ],

  fileInputRules: [
    (v: string) => !!v || "File is required",
    (v: string) => (v && v.length > 0) || "File is required",
  ],

  fieldNameRules: [
    (value: string) => {
      if (!value || value.trim()?.length < 0) {
        return "The name is required.";
      }
      if (/^(?:@(?:[a-z0-9-*~][a-z0-9-*._~]*)?\/)?[a-z0-9-~][a-z0-9-._~]*$/.test(value)) {
        return true;
      } else {
        return "The name format is not correct. It should not contain special characters, such as space, \\, *, etc. If you want to connect multiple words, use a hyphen (-) instead.";
      }
    },
  ],

  fieldVersionRules: [
    (value: string) => {
      if (!value || value.trim()?.length < 0) {
        return "The version is required.";
      }
      if (/^[0-9]+.[0-9]+.[0-9]+(?:-[a-z]+(?:[_\\.-]*[a-z0-9]+)*)*$/.test(value)) {
        return true;
      } else {
        return "The version format should follow semantic versioning.";
      }
    },
  ],
};

const alert = ref(false);
const projName = ref<string>();
const projVersion = ref("1.0.0");
const projDesc = ref("");
const projIsLibrary = ref(false);
const refUploadProgress = ref();
const selectedType = ref<AddProjectType | unknown>(AddProjectType.upload);
const projectFile = ref<File | null>();
const uploadIcon = ref<File[]>([]);

const showDependentArea = ref(false);
const showDependencySelect = ref(false);
const repoPath = ref("");
const dependentProjectNameSelected = ref<string[]>([]);
const searchResultsFromRegistry = ref<PackageSearchResult[]>([]);
const finalDependencies = ref<{ [k: string]: string }>({});
const tabValue = ref(null);
const formUpload = ref<HTMLFormElement>();
const formClone = ref<HTMLFormElement>();
const formRegistry = ref<HTMLFormElement>();

const root = computed(() => store.state.root);
const config = computed(() => store.state.projectConfig);
const selectedIcon = ref(config.value?.img);
const projectPool = computed(() => store.state.projectsPool);
const settings = computed(() => store.state.webinizerSettings);
const icons = computed(() => store.state.availableIcons);
const existUploadIcon = computed(
  () =>
    icons.value?.some((item) => item.isUploaded === true && selectedIcon.value?.name !== item.name)
);

const dependencySelectOptions = computed(() => {
  // local projects
  const projectNameArr: string[] = [];
  projectNameArr.push(
    ...projectPool.value.map((item) => {
      return localPrefixStr + String(item.name);
    })
  );
  // remote projects from registry
  if (searchResultsFromRegistry.value.length) {
    projectNameArr.push(
      ...searchResultsFromRegistry.value.map((item) => {
        return `${remotePrefixStr}${item.name}@${item.version}`;
      })
    );
  }
  return projectNameArr;
});

const registryPkgSearchList = computed(() => {
  const packageArr: string[] = [];

  if (searchResultsFromRegistry.value.length) {
    packageArr.push(
      ...searchResultsFromRegistry.value.map((item) => {
        return `${item.name}@${item.version}`;
      })
    );
  }
  return packageArr;
});

const projAddStatus = computed(() => store.state.projAddStatus);
const projDepUpdateStatus = computed(() => store.state.projDepUpdateStatus);
const dependentProjectNameToDisplay = computed(() => {
  const resultArr: DependentProjInfo[] = [];

  if (config.value?.dependencies) {
    Object.keys(config.value?.dependencies).forEach((dep) => {
      resultArr.push({
        name: dep,
        desc: config.value?.dependencies?.[dep] || "",
      });
    });
  }
  return resultArr;
});

const queryRegistryWithDebounce = lodash.debounce(queryRegistry, 500);

const prefix4ProjIcon = `${API_SERVER}/api/projects/${encodeURIComponent(root.value)}/icons/`;

function confirmPackageHandler(packageNameWithVersion: string) {
  // we should split the parameter to package name for `projName`
  // and package version for `version`
  if (packageNameWithVersion) {
    const splitResultArr = packageNameWithVersion.split(`@`);
    if (splitResultArr.length > 1) {
      projName.value = splitResultArr[0];
      projVersion.value = splitResultArr[1];
    } else {
      // if package name string does not include version info
      // regard it as project name directly
      projName.value = packageNameWithVersion;
    }
  }
}

function showAlert(idx: number) {
  deleteIdx = idx;
  alert.value = true;
}

function fileChangeHandler($event: Event) {
  const target = $event.target as HTMLInputElement;
  if (target !== undefined && target.files) {
    projectFile.value = target.files[0];
    const { type } = projectFile.value;
    if (type) {
      return;
    }
  }
}

function iconChangeHandler($event: Event) {
  // clear the selectedIcon value
  selectedIcon.value = undefined;
  const target = $event.target as HTMLInputElement;
  if (target !== undefined && target.files) {
    if (uploadIcon.value.length > 0) {
      uploadIcon.value[0] = target.files[0];
    } else {
      uploadIcon.value.push(target.files[0]);
    }
  }
}

/*
 *  bind project with icons in /src/assets/basic/preseticons
 *  according to the word initial of the project name
 *  the name of the icons is just the content,
 *  `0.png` means number `0`
 *  `a.png` means word `A` ...
 */
function bindIconAccordingToProjName(projectName: string) {
  const iconFileExtends = ".png";
  const matches = projectName.match(/[a-zA-Z0-9]/);
  if (matches) {
    return matches[0].toLocaleLowerCase() + iconFileExtends;
  }
  return "";
}

async function selectIconHandler(iconObj: ProjectIcon) {
  uploadIcon.value = [];
  selectedIcon.value = iconObj;

  if (root.value) await saveConfig({ img: iconObj });
}

async function saveConfig(configToMerge?: { [k: string]: unknown }) {
  try {
    if (configToMerge) {
      // store.commit("mergeProjectConfig", configToMerge);
      await store.dispatch("saveProjectConfig", configToMerge);
    } else {
      await store.dispatch("saveProjectConfig");
    }
  } catch (error) {
    log.errMsgNCuz(error);
  }
}

async function saveProjectName(name: string | undefined) {
  await saveConfig({ name });
}

async function saveProjectVersion(version: string | undefined) {
  await saveConfig({ version });
}

async function saveProjectDesc(desc: string | undefined) {
  await saveConfig({ desc });
}

async function saveProjectIsLib() {
  await saveConfig({ isLibrary: !config.value?.isLibrary });
}

async function queryRegistry(text: string | undefined) {
  // use case-insensitive search
  if (settings.value?.registry && text)
    searchResultsFromRegistry.value = await searchRegistry(text.toLocaleLowerCase());
  else searchResultsFromRegistry.value = [];
}

/**
 * NOTE - there are 2 scenarios:
 *        1. add dependencies when creating new project
 *           the dependencies should be saved and update
 *           when clicking `create` button.
 *        2. add dependencies for existed project
 *           update when clicking `finish` button.
 */
async function triggerAddDependencies() {
  //return if user does not input or select any dependency
  if (dependentProjectNameSelected.value.length === 0) {
    showDependencySelect.value = false;
    return;
  }

  const projectDependenciesToAdd: { [k: string]: string } = {};

  dependentProjectNameSelected.value.forEach((item) => {
    if (item.startsWith(localPrefixStr)) {
      // handle local dependencies
      const projectName = item.replace(localPrefixStr, "");
      const projIndex = projectPool.value.findIndex(
        (proj) => proj.name?.localeCompare(projectName, undefined, { sensitivity: "base" }) === 0
      );
      if (projIndex > -1 && projectPool.value[projIndex].path) {
        Object.assign(projectDependenciesToAdd, {
          // eslint-disable-next-line @typescript-eslint/restrict-template-expressions
          [projectName.toLocaleLowerCase()]: `file:${projectPool.value[projIndex].path}`,
        });
      }
    } else if (item.startsWith(remotePrefixStr)) {
      // handle remote dependencies
      const projectInfo = item.replace(remotePrefixStr, "");
      const [projectName, projectVersion] = projectInfo.split("@");
      if (projectName && projectVersion)
        Object.assign(projectDependenciesToAdd, { [projectName]: `^${projectVersion}` });
    } else {
      if (item.includes("@")) {
        const [projectName, projectVersion] = item.split("@");
        Object.assign(projectDependenciesToAdd, { [projectName]: `${projectVersion}` });
      } else {
        Object.assign(projectDependenciesToAdd, { [item]: `` });
      }
    }
  });

  const newDependenciesValue: { [k: string]: string } = Object.assign(
    {},
    config.value?.dependencies,
    projectDependenciesToAdd
  );

  if (!root.value) {
    finalDependencies.value = newDependenciesValue;
    return;
  }

  //clear the selection in box
  dependentProjectNameSelected.value = [];
  //hide the select box after selection
  showDependencySelect.value = false;
  await saveConfig({ dependencies: newDependenciesValue });
}

async function deleteDependencies() {
  try {
    if (!root.value || deleteIdx === -1) return;
    const projectDependenciesToDel = dependentProjectNameToDisplay.value[deleteIdx];
    const depsCopy = cloneDeep(config.value?.dependencies) || {};
    /* reuse desc to represent version temporarily*/
    if (
      Object.keys(depsCopy).includes(projectDependenciesToDel.name) &&
      depsCopy[projectDependenciesToDel.name] === projectDependenciesToDel.desc
    ) {
      delete depsCopy[projectDependenciesToDel.name];
    }
    await saveConfig({ dependencies: depsCopy });
    alert.value = false;
    deleteIdx = -1;
  } catch (error) {
    log.errMsgNCuz(error);
  }
}

async function uploadProjectFile() {
  let uploadedSize = 0;
  let chunkIndex = 0;
  // splice the projectFile
  if (projectFile.value) {
    try {
      const { name, size, type } = projectFile.value;
      const startTime = convertDate(new Date());
      while (uploadedSize < size) {
        let blob = null;
        if (uploadedSize + projectChunkSize > size) {
          blob = projectFile.value.slice(uploadedSize, size);
        } else {
          blob = projectFile.value.slice(uploadedSize, uploadedSize + projectChunkSize);
        }
        let blobFile = new File([blob], `${name}`);
        let formData = new FormData();
        formData.append("file", blobFile);
        formData.append("index", String(chunkIndex));
        formData.append("type", type);
        formData.append("chunkSize", String(projectChunkSize));
        formData.append("uploadedSize", String(uploadedSize));
        formData.append("totalSize", String(size));
        formData.append("startTime", String(startTime).replace(/ /g, `_`));
        formData.append("projectName", projName.value as string);
        formData.append("projectVersion", projVersion.value);
        formData.append("projectDesc", projDesc.value);
        formData.append("projectIsLib", String(projIsLibrary.value));
        formData.append("projectDependencies", JSON.stringify(finalDependencies.value, null, 2));
        formData.append(
          "img",
          JSON.stringify(
            selectedIcon.value || {
              name: bindIconAccordingToProjName(projName.value as string),
              isUploaded: false,
            },
            null,
            2
          )
        );

        await store.dispatch("uploadProjectFile", formData);
        uploadedSize += blobFile.size;
        refUploadProgress.value = (uploadedSize / size) * 100;
        chunkIndex++;
      }
    } catch (error) {
      store.commit("setProjAddStatus", ProjectAddStatus.done);
      log.errMsgNCuz(error);
      return;
    }
  }
}

async function uploadProjectIcon() {
  const { name } = uploadIcon.value[0];
  let blob = uploadIcon.value[0];
  let blobFile = new File([blob], `${name}`);
  let formData = new FormData();
  formData.append("file", blobFile);
  await store.dispatch("uploadProjectIcon", formData);

  uploadIcon.value = [];
  // refresh icons list
  await getAvailableIcons();
}

async function cloneProject() {
  await store.dispatch("cloneProjectFile", {
    img: selectedIcon.value || {
      name: bindIconAccordingToProjName(projName.value as string),
      isUploaded: false,
    },
    name: projName.value,
    version: projVersion.value,
    desc: projDesc.value,
    isLibrary: projIsLibrary.value,
    dependencies: finalDependencies.value,
    repoPath: repoPath.value,
  });
}

async function fetchProject() {
  await store.dispatch("fetchProjectFromRegistry", {
    name: projName.value,
    version: projVersion.value,
  });
}

async function submitProject() {
  switch (selectedType.value) {
    case AddProjectType.upload: {
      const { valid } = await formUpload.value?.validate();
      if (valid) await uploadProjectFile();
      break;
    }

    case AddProjectType.clone: {
      const { valid } = await formClone.value?.validate();
      if (valid) await cloneProject();
      break;
    }

    case AddProjectType.registry: {
      const { valid } = await formRegistry.value?.validate();
      if (valid) await fetchProject();
      break;
    }
    default:
      break;
  }
}

async function removeIcon(iconObj: ProjectIcon) {
  // only uploaded icons can be removed
  if (iconObj.isUploaded) {
    await store.dispatch("removeIcon", prefix4ProjIcon + iconObj.name);
  }
}

async function getAvailableIcons() {
  await store.dispatch("getAllAvailableIcons");
}

onMounted(async () => {
  try {
    await store.dispatch("fetchWebinizerSettings");
    if (route.query.root) {
      store.commit("setRoot", route.query.root);
    }
    if (root.value) await store.dispatch("fetchProjectConfig");
    await store.dispatch("fetchProjects");

    if (config.value?.dependencies && Object.keys(config.value.dependencies).length) {
      showDependentArea.value = true;
    }
  } catch (error) {
    log.errMsgNCuz(error);
  }
});

watch(
  () => projAddStatus.value,
  async (_n) => {
    if (_n === ProjectAddStatus.done) {
      if (root.value) {
        /*NOTE - reset dependencies selection box */
        if (Object.keys(finalDependencies.value).length !== 0) {
          dependentProjectNameSelected.value = [];
          showDependencySelect.value = false;
          showDependentArea.value = true;
        }

        /* upload the project icon after adding */
        if (uploadIcon.value.length > 0) {
          /* trigger to upload icon */
          await uploadProjectIcon();
        }

        /* update the selected icon after uploading*/
        await store.dispatch("fetchProjectConfig");

        /* if the config.img object is undefined, bind with default
         * icon based on work-initial
         */
        if (config.value?.img === undefined) {
          await saveConfig({
            img: {
              name: bindIconAccordingToProjName(projName.value as string),
              isUploaded: false,
            },
          });
        }

        toast.success(`Project ${projName.value as string} has been successfully added!`);
      }
    }
  }
);

watch(
  () => projDepUpdateStatus.value,
  (_n) => {
    if (_n === ProjectDepUpdateStatus.done) {
      if (Object.keys(finalDependencies.value).length !== 0) {
        dependentProjectNameSelected.value = [];
        showDependencySelect.value = false;
        showDependentArea.value = true;
      }
    }
  }
);

watch(
  () => config.value?.dependencies,
  (_n) => {
    if (_n) {
      if (config.value?.dependencies && Object.keys(config.value.dependencies).length)
        showDependentArea.value = true;
    }
  }
);

watch(
  () => config.value?.img,
  (_n) => {
    if (_n) {
      if (config.value?.img && Object.keys(config.value?.img).length > 0) {
        selectedIcon.value = config.value?.img;
      }
    }
  }
);
</script>

<style lang="scss" scoped>
:deep(.v-list) {
  margin: 0;
  padding: 0 8px !important;
}

:deep(.v-list-item) {
  min-height: auto !important;
  padding: 0px 8px !important;
}

:deep(.v-list-item-title) {
  font-size: 0.8rem !important;
}

.subtitle-2 {
  font-size: 1rem;
}
.v-btn--active .btn_slider {
  opacity: 1;
}
.btn_slider {
  position: absolute;
  bottom: 0;
  left: 0;
  height: 2px;
  width: 100%;
  background: currentColor;
  pointer-events: none;
  opacity: 0;
}

.modulelink {
  color: var(--p-6);
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
  direction: rtl;
}

.modulelink:hover {
  color: var(--p);
  text-decoration: underline;
  cursor: pointer;
}
</style>
