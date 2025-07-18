<script setup lang="ts">
const { t } = useI18n()
const localePath = useLocalePath()
const router = useRouter()
const toast = useToast()
const isModernFileAPISupport = ref(true)
const isDirSupport = ref(true)
const receiveCode = ref('')
const isFileDraging = ref(false)
const fileDragArea = ref()

const { data: transCount } = useFetch('/api/transCount', {
  method: 'post',
  onResponseError() {
    toast.add({
      severity: 'error',
      summary: 'Error',
      detail: t('hint.serviceUnavailable')
    })
  }
})

useSeoMeta({
  title: t('home')
})

// 发起目录同步
function syncDir() {
  // if (isModernFileAPISupport.value) {
  // showDirectoryPicker()
  //   .then((fh) => {
  //     useFullScreenLoader(true)
  //     return dealFilesFromHandler(fh)
  //   })
  //   .then((fileMap) => {
  //     if (Object.keys(fileMap).length === 0) {
  //       throw '目录为空'
  //     }
  //     useFilesInfo('syncDir', fileMap)
  //     router.push(localePath('/sender'))
  //     useFullScreenLoader(false)
  //   })
  //   .catch((e) => {
  //     console.warn(e)
  //     toast.add({ severity: 'error', summary: 'Error', detail: e, life: 3000 })
  //     useFullScreenLoader(false)
  //   })
  // } else {
  // 注意：移动端不支持选择目录
  selectDir()
    .then((files) => {
      useFullScreenLoader(true)
      return dealFilesFormList(files)
    })
    .then((fileMap) => {
      if (Object.keys(fileMap).length === 0) {
        throw 'The directory is empty'
      }
      // console.log(fileMap)
      useFilesInfo('syncDir', fileMap)
      router.push(localePath('/sender'))
    })
    .catch((e) => {
      console.warn(e)
      toast.add({ severity: 'error', summary: 'Error', detail: e, life: 5e3 })
      useFullScreenLoader(false)
    })
  // }
}

// 发送目录
function sendDir() {
  // 注意：移动端不支持选择目录
  selectDir()
    .then((files) => {
      useFullScreenLoader(true)
      return dealFilesFormList(files)
    })
    .then((fileMap) => {
      if (Object.keys(fileMap).length === 0) {
        throw '目录为空'
      }
      useFilesInfo('transDir', fileMap)
      router.push(localePath('/sender'))
    })
    .catch((e) => {
      console.warn(e)
      toast.add({ severity: 'error', summary: 'Error', detail: e, life: 5e3 })
      useFullScreenLoader(false)
    })
}

// 发送文件
function sendFile() {
  selectFile()
    .then((file) => {
      useFullScreenLoader(true)
      useFilesInfo('transFile', dealFilesFormFile(file))
      router.push(localePath('/sender'))
    })
    .catch((e) => {
      console.warn(e)
      toast.add({ severity: 'error', summary: 'Error', detail: e, life: 5e3 })
      useFullScreenLoader(false)
    })
}

watch(isFileDraging, (val) => {
  if (val) {
    fileDragArea.value.style.display = 'flex'
    fileDragArea.value.style.opacity = '1'
  } else {
    fileDragArea.value.style.opacity = '0'
    setTimeout(() => {
      if (fileDragArea.value) {
        fileDragArea.value.style.display = 'none'
      }
    }, 300)
  }
})

function fileDragOver(e: Event) {
  e.preventDefault()
  isFileDraging.value = true
}

function fileDrop(e: DragEvent) {
  e.preventDefault()
  isFileDraging.value = false
  // console.log('fileDrop', e)

  if (e.dataTransfer && e.dataTransfer.items.length > 0) {
    const item = e.dataTransfer.items[0].webkitGetAsEntry()
    const files = e.dataTransfer.files
    if (item) {
      if (item.isFile) {
        const file = files[0]
        useFullScreenLoader(true)
        useFilesInfo('transFile', dealFilesFormFile(file))
        router.push(localePath('/sender'))
      } else if (item.isDirectory) {
        toast.add({
          severity: 'warn',
          summary: 'Warn',
          detail: t('hint.noSupportFolderDrag'),
          life: 5e3
        })
      }
    }
  }
}

watch(
  receiveCode,
  () => {
    if (receiveCode.value.length === 4) {
      if (/^\d{4}$/.test(receiveCode.value)) {
        useFullScreenLoader(true)
        router.push({ path: localePath('recipient'), query: { code: receiveCode.value } })
      } else {
        receiveCode.value = receiveCode.value.replaceAll(' ', '')
      }
    }
    // console.log(inputCode.value)
  },
  { flush: 'sync' }
)

onMounted(() => {
  isModernFileAPISupport.value = isModernFileAPIAvailable()
  isDirSupport.value = supportsDirectorySelection()
  useFilesInfo('', {})

  window.ondragenter = (e) => {
    e.preventDefault()
    isFileDraging.value = true
  }
  window.ondragleave = (e: DragEvent) => {
    e.preventDefault()
    if (!e.relatedTarget) {
      isFileDraging.value = false
    }
  }
  window.ondragover = (e) => {
    e.preventDefault()
  }
  window.ondrop = (e) => {
    e.preventDefault()
    isFileDraging.value = false
  }
})
</script>

<template>
  <div class="flex flex-col min-h-screen overflow-hidden bg-slate-900">
    <!-- Site header -->
    
    <!-- Page content -->
    <main class="grow">
      <section class="relative">

        <!-- Illustration -->
        <div class="hidden md:block absolute left-1/2 -translate-x-1/2 pointer-events-none -z-10" aria-hidden="true">
          <img src="@/assets/images/hero-illustration.svg" class="max-w-none" width="1440" height="1265" alt="Hero Illustration">
        </div>
         <div class="relative max-w-6xl mx-auto px-4 sm:px-6">
      <div class="pt-32 md:pt-40">

        <!-- Hero content -->
        <div class="max-w-3xl mx-auto text-center">

          <h1 class="h1 font-hkgrotesk mb-6" data-aos="fade-up">A powerful suite of user-centric products</h1>
          <p class="text-xl text-slate-500 mb-10" data-aos="fade-up" data-aos-delay="100">Our landing page template works on all devices, so you only have to set it up once, and get beautiful results forever.</p>
          <div class="max-w-xs mx-auto sm:max-w-none sm:inline-flex sm:justify-center space-y-4 sm:space-y-0 sm:space-x-4" data-aos="fade-up" data-aos-delay="200">
            <div>
              <router-link class="btn text-white bg-indigo-500 hover:bg-indigo-600 w-full shadow-xs group" to="/signup">
                Get Started Free <span class="tracking-normal text-sky-300 group-hover:translate-x-0.5 transition-transform duration-150 ease-in-out ml-1">-&gt;</span>
              </router-link>
            </div>
            <div>
              <a class="btn text-slate-300 bg-slate-700 hover:bg-slate-600 border-slate-600 w-full shadow-xs" href="#0">Read Docs</a>
            </div>
          </div>

        </div>

        <!-- Hero image -->
        <div class="pt-16 md:pt-20" data-aos="fade-up" data-aos-delay="300">
          <img class="mx-auto" src="@/assets/images/hero-image.png" width="920" height="518" alt="Hero">
        </div>

      </div>
    </div>
      </section>
    </main>
    
    <!-- Site footer -->
  </div>
</template>

<style>
.file-drag-area {
  display: none;
  opacity: 0;
  transition-property: opacity;
  transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1);
  transition-duration: 300ms;
}
</style>
