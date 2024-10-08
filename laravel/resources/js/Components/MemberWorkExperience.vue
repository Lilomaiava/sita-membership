<script setup>
import { Link, useForm } from '@inertiajs/vue3'
import { FwbAlert, FwbButton } from 'flowbite-vue'
import { ref, nextTick, onMounted } from 'vue'
import InputLabel from './InputLabel.vue'
import Input from './Input.vue'
import PencilOutlineIcon from 'vue-material-design-icons/PencilOutline.vue'
import InputError from '@/Components/InputError.vue'
import DeleteConfirmationModal from '@/Components/DeleteConfirmationModal.vue'
import DialogModal from '@/Components/DialogModal.vue'
import DeleteButton from '@/Components/DeleteButton.vue'
import CancelButton from '@/Components/CancelButton.vue'

const props = defineProps({
  memberId: Number,
  memberWorkExperiences: Object,
})
const isShowModal = ref(false)
const showConfirmationModal = ref(false)
const organisation = ref(null)

const form = useForm({
  id: null,
  member_id: props.memberId,
  organisation: null,
  position: null,
  responsibilities: null,
  from_date: null,
  to_date: null,
  is_current: false,
})

function closeModal() {
  isShowModal.value = false
  form.clearErrors()
}
function showModal(workExperience) {
  isShowModal.value = true

  nextTick(() => {
    form.id = workExperience.id
    form.member_id = props.memberId
    form.organisation = workExperience.organisation
    form.position = workExperience.position
    form.responsibilities = workExperience.responsibilities
    form.from_date = workExperience.from_date
    form.to_date = workExperience.to_date
    form.is_current = workExperience.is_current
    organisation.value.focus()
  })
}
function addSaveWorkExperience() {
  if (!form.id) {
    form.post('/member-work-experiences', {
      preserveScroll: true,
      onSuccess: () => {
        form.reset()
        closeModal()
      },
    })
  } else {
    form.put('/member-work-experiences/' + form.id, {
      preserveScroll: true,
      onSuccess: () => {
        form.reset()
        closeModal()
      },
    })
  }
}

function deleteItem() {
  form.delete('/member-work-experiences/' + form.id, {
    preserveScroll: true,
  })
  showConfirmationModal.value = false
  closeModal()
}
</script>
<template>
  <div>
    <h5>Work Experience</h5>

    <Link href="#">
      <fwb-Button class="p-3 mt-3" color="alternative" @click.prevent="showModal">Add Work Experience</fwb-Button>
    </Link>

    <div v-if="memberWorkExperiences ? memberWorkExperiences.length : false" class="relative mt-3 overflow-x-auto shadow-md sm:rounded-lg">
      <table class="w-full text-sm text-left text-gray-500 dark:text-gray-400">
        <thead class="text-xs text-gray-700 uppercase bg-gray-50 dark:bg-gray-700 dark:text-gray-400">
          <tr>
            <th scope="col" class="px-6 py-3">
              <span class="sr-only">Edit</span>
            </th>
            <th scope="col" class="px-6 py-3">Organisation</th>
            <th scope="col" class="px-6 py-3">Position</th>
            <th scope="col" class="px-6 py-3">Responsibilities</th>
            <th scope="col" class="px-6 py-3">Date</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="workExperience in memberWorkExperiences" class="bg-white border-b dark:bg-gray-800 dark:border-gray-700">
            <td class="px-6 py-4 text-right">
              <fwb-button type="button" class="font-medium text-blue-600 dark:text-blue-500 hover:underline" @click="showModal(workExperience)"><PencilOutlineIcon fill-color="green" /></fwb-button>
            </td>
            <th scope="row" class="px-6 py-4 font-medium text-gray-900 dark:text-white whitespace-nowrap">
              {{ workExperience.organisation }}
            </th>
            <td class="px-6 py-4">{{ workExperience.position }}</td>
            <td class="px-6 py-4">{{ workExperience.responsibilities }}</td>
            <td class="px-6 py-4 whitespace-nowrap">
              {{ workExperience.from_date }}<span v-if="!workExperience.is_current"> to {{ workExperience.to_date }}</span> <span v-else> to Current</span>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>

  <!-- Modal -->
  <DialogModal :show="isShowModal" @close="closeModal">
    <template #title>
      <div v-if="!form.id" class="flex items-center text-lg">Add Work Experience</div>
      <div v-else class="flex items-center text-lg">Edit Work Experience</div>
    </template>
    <template #content>
      <!-- Organisation -->
      <InputLabel for="organisation" value="Organisation" class="mb-2 required" />
      <Input id="organisation" ref="organisation" v-model="form.organisation" type="text" placeholder="enter your organisation" />
      <InputError class="mt-2 mb-4" :message="form.errors.organisation" />

      <!-- Position -->
      <InputLabel for="position" value="Position" class="mb-2 required" />
      <Input id="position" v-model="form.position" placeholder="enter your position" label="Position" class="mb-2" />
      <InputError class="mt-2 mb-4" :message="form.errors.position" />

      <!-- Responsibilities -->
      <fwb-Alert type="info" class="mt-3 mb-2">Please provide only a brief summary of roles and responsibilities. </fwb-Alert>
      <InputLabel for="message" value="Responsibilities" class="mb-2 required" />
      <textarea id="message" v-model="form.responsibilities" rows="4" class="block p-2.5 w-full text-sm text-gray-900 bg-gray-50 rounded-lg border border-gray-300 focus:ring-blue-500 focus:border-blue-500 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500 mb-2" placeholder="Please provide only a brief summary of roles and responsibilities..." />
      <InputError class="mt-2 mb-4" :message="form.errors.responsibilities" />

      <!-- Is Current -->
      <div class="relative max-w-sm mb-3">
        <input id="is_current" v-model="form.is_current" :checked="form.is_current" type="checkbox" class="w-4 h-4 text-blue-600 bg-gray-100 border-gray-300 rounded focus:ring-blue-500 dark:focus:ring-blue-600 dark:ring-offset-gray-800 focus:ring-2 dark:bg-gray-700 dark:border-gray-600" placeholder="Select date" />
        <label for="is_current" class="ml-2 ms-2 text-sm font-medium text-gray-900 dark:text-gray-300">Currently employed here</label>
      </div>
      <InputError class="mt-2 mb-4" :message="form.errors.is_current" />

      <!-- From Date -->
      <InputLabel for="from_date" value="From Date" class="mb-2 required" />
      <div class="relative max-w-sm mb-3">
        <input id="from_date" v-model="form.from_date" type="date" class="bg-gray-50 border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-full p-2.5 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500" placeholder="Select date" />
      </div>
      <InputError class="mt-2 mb-4" :message="form.errors.from_date" />

      <!-- To Date -->
      <span v-show="!form.is_current">
        <InputLabel for="to_date" value="To Date" class="mb-2 required" />
        <div class="relative max-w-sm mb-3">
          <input id="to_date" v-model="form.to_date" type="date" class="bg-gray-50 border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-full p-2.5 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500" placeholder="Select date" />
        </div>
        <InputError class="mt-2 mb-4" :message="form.errors.to_date" />
      </span>
    </template>
    <template #footer>
      <div>
        <CancelButton @click="closeModal" />
      </div>
      <div>
        <DeleteButton v-if="form.id" @click="showConfirmationModal = true" />
        <fwb-button type="submit" :disabled="form.processing" class="ml-2 text-white bg-blue-700 hover:bg-blue-800 focus:ring-4 focus:outline-none focus:ring-blue-300 font-medium rounded-lg text-sm px-5 py-2.5 text-center dark:bg-blue-600 dark:hover:bg-blue-700 dark:focus:ring-blue-800" @click="addSaveWorkExperience">
          {{ !form.id ? 'Add' : 'Update' }}
        </fwb-button>
      </div>
    </template>
  </DialogModal>

  <DeleteConfirmationModal :show="showConfirmationModal" @delete="deleteItem" @close="showConfirmationModal = false" />
</template>
