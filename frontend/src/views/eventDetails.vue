
<!-- This component allows users to view and update an event,
     and editors to manage event attendees. -->

<template>
  <main>
    <div>
      <!-- Header -->
      <h1
        class="font-bold text-4xl text-red-700 tracking-widest text-center mt-10"
      >
        Event Details
      </h1>
    </div>

    <div class="px-10 py-20">

      <!-- ================================================== -->
      <!-- EVENT DETAILS -->
      <!-- ================================================== -->

      <div
        class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-4 gap-x-6 gap-y-10"
      >
        <h2 class="text-2xl font-bold">
          Event Details
        </h2>

        <!-- Event Name -->
        <div class="flex flex-col">
          <label class="block">
            <span class="text-gray-700">
              Event Name
            </span>
            <span style="color: #ff0000">*</span>

            <input
              v-model="event.name"
              type="text"
              class="w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
            />

            <span
              v-if="v$.event.name.$error"
              class="text-red-500"
            >
              Event Name is required
            </span>
          </label>
        </div>

        <!-- Date -->
        <div class="flex flex-col">
          <label class="block">
            <span class="text-gray-700">
              Date
            </span>
            <span style="color: #ff0000">*</span>

            <input
              v-model="event.date"
              type="date"
              class="w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
            />

            <span
              v-if="v$.event.date.$error"
              class="text-red-500"
            >
              <span
                v-if="v$.event.date.required.$invalid"
              >
                Event Date is required
              </span>

              <span
                v-else-if="
                  !v$.event.date.required.$invalid &&
                  v$.event.date.validDate.$invalid
                "
              >
                Event Date must be a valid date
              </span>
            </span>
          </label>
        </div>

        <div></div>
        <div></div>

        <!-- Description -->
        <div class="flex col-span-2 flex-col">
          <label class="block">
            <span class="text-gray-700">
              Description
            </span>

            <textarea
              v-model="event.description"
              class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
              rows="2"
            ></textarea>
          </label>
        </div>
      </div>


      <!-- ================================================== -->
      <!-- SERVICES OFFERED -->
      <!-- ================================================== -->

      <div
        class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-4 gap-x-6 gap-y-10 mt-10"
      >
        <h2 class="text-2xl font-bold">
          Services Offered at Event
        </h2>

        <div class="flex flex-col grid-cols-3">
          <div>
            <ul
              v-if="services.length"
              class="space-y-2"
            >
              <li
                v-for="service in services"
                :key="service._id"
                :data-service-id="service._id"
                class="rounded-lg border border-gray-300 p-2 hover:bg-gray-100 transition-colors relative"
              >
                <label class="block w-full h-full">
                  <input
                    v-model="event.services"
                    type="checkbox"
                    :id="service._id"
                    :value="service._id"
                    :disabled="service.status === 'Inactive'"
                    class="rounded border-gray-300 disabled:opacity-50 text-indigo-600 shadow-sm focus:border-indigo-300 focus:ring focus:ring-offset-0 focus:ring-indigo-200 focus:ring-opacity-50 mr-2"
                  />

                  <span class="font-medium">
                    {{ service.name }}
                  </span>
                </label>
              </li>
            </ul>

            <p
              v-else
              class="text-gray-600"
            >
              No Active Services Available
            </p>
          </div>
        </div>
      </div>


      <!-- ================================================== -->
      <!-- ADDRESS -->
      <!-- ================================================== -->

      <div
        class="mt-10 grid grid-cols-1 sm:grid-cols-2 md:grid-cols-4 gap-x-6 gap-y-10"
      >
        <h2 class="text-2xl font-bold">
          Address
        </h2>

        <div class="flex flex-col">
          <label class="block">
            <span class="text-gray-700">
              Address Line 1
            </span>

            <input
              v-model="event.address.line1"
              type="text"
              class="w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
            />
          </label>
        </div>

        <div class="flex flex-col">
          <label class="block">
            <span class="text-gray-700">
              Address Line 2
            </span>

            <input
              v-model="event.address.line2"
              type="text"
              class="w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
            />
          </label>
        </div>

        <div class="flex flex-col">
          <label class="block">
            <span class="text-gray-700">
              City
            </span>

            <input
              v-model="event.address.city"
              type="text"
              class="w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
            />
          </label>
        </div>

        <div></div>

        <div class="flex flex-col">
          <label class="block">
            <span class="text-gray-700">
              County
            </span>

            <input
              v-model="event.address.county"
              type="text"
              class="w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
            />
          </label>
        </div>

        <div class="flex flex-col">
          <label class="block">
            <span class="text-gray-700">
              Zip Code
            </span>

            <input
              v-model="event.address.zip"
              type="text"
              class="w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
            />
          </label>
        </div>
      </div>


      <!-- ================================================== -->
      <!-- EVENT ACTION BUTTONS -->
      <!-- ================================================== -->

      <div
        class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-4 gap-x-6 gap-y-10"
      >
        <div></div>

        <div class="flex justify-between mt-10 mr-20">
          <button
            type="button"
            class="bg-green-700 disabled:opacity-50 text-white rounded"
            :disabled="user.role === 'viewer'"
            @click="submitEventUpdate"
          >
            Update Event
          </button>
        </div>

        <div class="flex justify-between mt-10 mr-20">
          <button
            type="button"
            class="bg-red-700 disabled:opacity-50 text-white rounded"
            :disabled="user.role === 'viewer'"
            @click="submitDeleteEvent"
          >
            Delete Event
          </button>
        </div>

        <div class="flex justify-between mt-10 mr-20">
          <button
            type="button"
            class="border border-red-700 bg-white text-red-700 rounded"
            @click="router.back()"
          >
            Go back
          </button>
        </div>
      </div>

      <hr class="mt-10 mb-10" />


      <!-- ================================================== -->
      <!-- ATTENDEE REGISTRATION AND DIRECTORY -->
      <!-- ================================================== -->

      <section
        class="rounded-2xl border border-gray-200 bg-white p-6 shadow-sm"
      >

        <!-- Section Heading -->
        <div
          class="mb-6 flex flex-wrap items-start justify-between gap-3"
        >
          <div>
            <h2
              class="text-2xl font-bold text-slate-900"
            >
              Event Attendees
            </h2>

            <p class="mt-1 text-sm text-slate-500">
              {{ clients.length }}
              registered
              {{ clients.length === 1 ? 'client' : 'clients' }}.
              Click an attendee's name to view their details.
            </p>
          </div>

          <span
            class="rounded-full bg-red-50 px-4 py-2 text-sm font-semibold text-red-700"
          >
            {{ clients.length }} Registered
          </span>
        </div>


        <!-- ================================================== -->
        <!-- REGISTER AN EXISTING CLIENT - EDITOR ONLY -->
        <!-- ================================================== -->

        <div
          v-if="user.role === 'editor'"
          class="mb-8 rounded-xl border border-red-100 bg-slate-50 p-5"
        >

          <h3
            class="text-lg font-bold text-slate-900"
          >
            Register Existing Client
          </h3>

          <p
            class="mt-1 text-sm text-slate-600"
          >
            Choose a client already saved in the database.
            Registered clients are excluded from the list.
          </p>

          <div
            class="mt-4 grid gap-3 md:grid-cols-[1fr_1fr_auto] md:items-end"
          >

            <!-- Search Existing Clients -->
            <label class="block">
              <span
                class="mb-2 block text-sm font-semibold text-slate-700"
              >
                Search Clients
              </span>

              <input
                v-model="clientSearch"
                type="search"
                placeholder="Search by name or city"
                class="w-full rounded-xl border border-slate-300 bg-white px-4 py-3 text-slate-900 focus:border-red-500 focus:outline-none focus:ring-2 focus:ring-red-200"
              />
            </label>


            <!-- Select Existing Client -->
            <label class="block">
              <span
                class="mb-2 block text-sm font-semibold text-slate-700"
              >
                Select Client
              </span>

              <select
                v-model="selectedClientId"
                :disabled="
                  registrationBusy ||
                  !availableClients.length
                "
                class="w-full rounded-xl border border-slate-300 bg-white px-4 py-3 text-slate-900 focus:border-red-500 focus:outline-none focus:ring-2 focus:ring-red-200 disabled:opacity-60"
              >
                <option value="">
                  Choose a client
                </option>

                <option
                  v-for="client in availableClients"
                  :key="client._id"
                  :value="client._id"
                >
                  {{ client.firstName }}
                  {{ client.lastName }}
                  —
                  {{ client.address?.city || 'City not provided' }}
                </option>
              </select>
            </label>


            <!-- Register Button -->
            <button
              type="button"
              :disabled="
                registrationBusy ||
                !selectedClientId
              "
              class="rounded-xl bg-red-700 px-5 py-3 font-bold text-white transition hover:bg-red-600 disabled:cursor-not-allowed disabled:opacity-50"
              @click="addAttendee"
            >
              {{
                registrationBusy
                  ? 'Saving...'
                  : '+ Register Client'
              }}
            </button>
          </div>

          <p
            v-if="!availableClients.length"
            class="mt-3 text-sm text-slate-500"
          >
            No unregistered clients match this search.
          </p>
        </div>


        <!-- ================================================== -->
        <!-- REGISTERED ATTENDEES TABLE -->
        <!-- ================================================== -->

        <div
          class="overflow-x-auto rounded-xl border border-slate-200"
        >
          <table
            class="min-w-full divide-y divide-slate-200 text-left"
          >
            <thead
              class="bg-slate-50 text-sm font-bold text-slate-900"
            >
              <tr>
                <th class="p-4">
                  Name
                </th>

                <th class="p-4">
                  City
                </th>

                <th class="p-4">
                  Phone Number
                </th>

                <th
                  v-if="user.role === 'editor'"
                  class="p-4 text-right"
                >
                  Actions
                </th>
              </tr>
            </thead>

            <tbody
              class="divide-y divide-slate-200 text-sm"
            >

              <!-- Registered Attendees -->
              <tr
                v-for="client in clients"
                :key="client._id"
                class="hover:bg-slate-50"
              >

                <td class="p-4">
                  <button
                    type="button"
                    class="font-semibold text-red-700 hover:underline"
                    @click="goToClientDetails(client._id)"
                  >
                    {{ client.firstName }}
                    {{ client.lastName }}
                  </button>
                </td>

                <td class="p-4">
                  {{ client.address?.city || '—' }}
                </td>

                <td class="p-4">
                  {{ client.phoneNumber?.primary || '—' }}
                </td>


                <!-- Remove Registration - Editor Only -->
                <td
                  v-if="user.role === 'editor'"
                  class="p-4 text-right"
                >
                  <button
                    type="button"
                    :disabled="registrationBusy"
                    class="rounded-lg border border-red-200 px-3 py-2 font-semibold text-red-700 hover:bg-red-50 disabled:opacity-50"
                    @click="removeAttendee(client)"
                  >
                    Remove
                  </button>
                </td>
              </tr>


              <!-- Empty State -->
              <tr v-if="!clients.length">
                <td
                  :colspan="
                    user.role === 'editor'
                      ? 4
                      : 3
                  "
                  class="p-8 text-center text-slate-500"
                >
                  No clients are registered for this event yet.
                </td>
              </tr>

            </tbody>
          </table>
        </div>

      </section>
    </div>
  </main>
</template>


<script setup>
import {
  reactive,
  ref,
  computed,
  onMounted
} from 'vue'

import {
  useRoute,
  useRouter
} from 'vue-router'

import useVuelidate from '@vuelidate/core'
import { required } from '@vuelidate/validators'
import { useToast } from 'vue-toastification'
import { useLoggedInUserStore } from '../store/loggedInUser'

import {
  getEventById,
  getEventAttendees,
  getClients,
  registerAttendee,
  deregisterAttendee,
  getServices,
  updateEvent,
  deleteEvent
} from '../api/api'


// ======================================================
// INITIAL SETUP
// ======================================================

const route = useRoute()
const router = useRouter()
const toast = useToast()
const user = useLoggedInUserStore()


// ======================================================
// REACTIVE STATE
// ======================================================

const clients = ref([])
const services = ref([])
const allClients = ref([])

const clientSearch = ref('')
const selectedClientId = ref('')
const registrationBusy = ref(false)


// ======================================================
// AVAILABLE CLIENTS
// Excludes clients already registered for the event.
// Also supports searching by name or city.
// ======================================================

const availableClients = computed(() => {
  const registeredIds = new Set(
    clients.value.map(
      (client) => String(client._id)
    )
  )

  const query = clientSearch.value
    .trim()
    .toLowerCase()

  return allClients.value.filter((client) => {

    if (
      registeredIds.has(
        String(client._id)
      )
    ) {
      return false
    }

    const searchable = `
      ${client.firstName || ''}
      ${client.lastName || ''}
      ${client.address?.city || ''}
    `.toLowerCase()

    return searchable.includes(query)
  })
})


// ======================================================
// EVENT STATE
// ======================================================

const event = reactive({
  _id: null,
  name: '',
  description: '',
  date: '',
  address: {
    line1: '',
    line2: '',
    city: '',
    county: '',
    zip: ''
  },
  attendees: [],
  services: []
})


// ======================================================
// VALIDATION
// ======================================================

const validDate = (value) => {
  const date = new Date(value)
  return !isNaN(date)
}

const rules = {
  event: {
    name: {
      required
    },
    date: {
      required,
      validDate
    }
  }
}

const v$ = useVuelidate(
  rules,
  { event }
)


// ======================================================
// CLIENT NAVIGATION
// ======================================================

function goToClientDetails(id) {
  router.push({
    name: 'clientdetails',
    params: { id }
  })
}


// ======================================================
// REFRESH REGISTERED ATTENDEES
// ======================================================

async function refreshAttendees() {
  const attendees = await getEventAttendees(
    route.params.id
  )

  clients.value = attendees

  // Update local event attendee references
  event.attendees = attendees.map(
    (client) => client._id
  )
}


// ======================================================
// REGISTER CLIENT FOR EVENT
// ======================================================

async function addAttendee() {
  if (
    user.role !== 'editor' ||
    !selectedClientId.value ||
    registrationBusy.value
  ) {
    return
  }

  registrationBusy.value = true

  try {
    const result = await registerAttendee(
      route.params.id,
      selectedClientId.value
    )

    await refreshAttendees()

    toast.success(
      result?.message ||
      'Client registered successfully'
    )

    selectedClientId.value = ''
    clientSearch.value = ''

  } catch (error) {
    toast.error(
      error.message ||
      'Could not register client'
    )

  } finally {
    registrationBusy.value = false
  }
}


// ======================================================
// REMOVE CLIENT FROM EVENT
// ======================================================

async function removeAttendee(client) {
  if (
    user.role !== 'editor' ||
    registrationBusy.value
  ) {
    return
  }

  const confirmed = window.confirm(
    `Remove ${client.firstName} ${client.lastName} from this event?`
  )

  if (!confirmed) {
    return
  }

  registrationBusy.value = true

  try {
    const result = await deregisterAttendee(
      route.params.id,
      client._id
    )

    await refreshAttendees()

    toast.success(
      result?.message ||
      'Client removed from event'
    )

  } catch (error) {
    toast.error(
      error.message ||
      'Could not remove client'
    )

  } finally {
    registrationBusy.value = false
  }
}


// ======================================================
// UPDATE EVENT
// ======================================================

async function submitEventUpdate() {
  await v$.value.$validate()

  if (v$.value.$error) {
    return
  }

  try {
    const response = await updateEvent(
      route.params.id,
      event
    )

    toast.success(response)
    router.push('/findevents')

  } catch (error) {
    toast.error('Error updating event')
    console.error(error)
  }
}


// ======================================================
// DELETE EVENT
// ======================================================

async function submitDeleteEvent() {
  try {
    if (clients.value.length !== 0) {
      toast.info(
        'Event can not be deleted since it has attendees.'
      )
      return
    }

    const response = await deleteEvent(
      route.params.id
    )

    toast.success(response)
    router.push('/findevents')

  } catch (error) {
    toast.error(error)
  }
}


// ======================================================
// LOAD EVENT DATA
// ======================================================

onMounted(async () => {
  try {
    const [
      eventResponse,
      clientsResponse,
      servicesResponse,
      allClientsResponse
    ] = await Promise.all([
      getEventById(route.params.id),
      getEventAttendees(route.params.id),
      getServices(),
      user.role === 'editor'
        ? getClients()
        : Promise.resolve([])
    ])

    eventResponse.date = new Date(
      eventResponse.date
    )
      .toISOString()
      .substring(0, 10)

    Object.assign(
      event,
      eventResponse
    )

    clients.value = clientsResponse
    services.value = servicesResponse
    allClients.value = allClientsResponse

    event.attendees = clientsResponse.map(
      (client) => client._id
    )

  } catch (error) {
    toast.error('Error loading event data')
    console.error(error)
  }
})
</script>
