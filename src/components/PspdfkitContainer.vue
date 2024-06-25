<script setup>
import PSPDFKit, { ToolbarItem, Rect } from 'pspdfkit'
import { onMounted, onBeforeUnmount } from 'vue'

const emit = defineEmits(['loaded'])

async function loadPSPDFKit () {
  PSPDFKit.unload('.pdf-container')

  return PSPDFKit.load({
    document: '/dist/assets/iks.pdf',
    container: '.pdf-container',
    useIframe: true,
    initialViewState: new PSPDFKit.ViewState({
      showToolbar: true,
      formDesignMode: true
    }),
    baseUrl: window.location.origin + '/',
    licenseKey: ''
  })
}


onMounted(() => {
  loadPSPDFKit().then((instance) => {
    emit('loaded')

    let pagesArray = instance.contentDocument.querySelectorAll('.PSPDFKit-Spread')
    const container = instance.contentDocument.querySelector('.PSPDFKit-Root')
    const toolbar = instance.contentDocument.querySelector('.PSPDFKit-Toolbar')
    let toolIsOn = false
    let annoPosition
    let cursor
    let widgetWidth
    let widgetHeight
    let widgetType
    let elIndex = 0

    const assignEventListenersToPagesArray = () => {
      pagesArray.forEach((page) => {
        const pageIndex = Number(page.dataset.spreadIndex)

        page.addEventListener('mousemove', (e) => {
          const containerTop = container?.getBoundingClientRect().top
          const pageTop = page.getBoundingClientRect().top
          const pageOffestTop = page.offsetTop
          let topDifference = 0

          if (containerTop !== undefined) {
            topDifference = containerTop - pageTop
          }

          const preTransformedRect = new PSPDFKit.Geometry.Rect({
            left: e.pageX,
            top: e.pageY,
            width: widgetWidth,
            height: widgetHeight
          })

          const containerLeft = container?.getBoundingClientRect().left
          const pageLeft = page.getBoundingClientRect().left
          let leftDifference = 0

          if (containerLeft !== undefined) {
            leftDifference = containerLeft - pageLeft
          }

          if (toolIsOn) {
            const mousePositionLeft = e.clientX - pageLeft
            const mousePositionTop = e.clientY - pageTop

            annoPosition = instance.transformContentClientToPageSpace(
              preTransformedRect,
              pageIndex
            )

            cursor.style.transform = `translate(${ mousePositionLeft }px,
                ${ mousePositionTop + pageOffestTop }px)
                `
            cursor.style.width =
              preTransformedRect.width * instance.currentZoomLevel + 'px'
            cursor.style.height =
              preTransformedRect.height * instance.currentZoomLevel + 'px'

            cursor.style.border = '2px solid rgba(69, 55, 222, 1)'
            cursor.style.backgroundColor = 'rgba(69, 55, 222, 0.3)'
            cursor.style.position = 'fixed'
            cursor.style.top = 0
            cursor.style.left = 0
          }
        })

        page.addEventListener('mouseenter', () => {
          if (toolIsOn) {
            cursor = document.createElement('div')
            page.appendChild(cursor)
            cursor.classList.add('tool-cursor')
            elIndex = pageIndex
          }
        })

        page.addEventListener('mouseleave', () => {
          if (toolIsOn) {
            instance.contentDocument
              .querySelectorAll('.tool-cursor')
              ?.forEach((el) => {
                el.remove()
              })
          }
        })

        page.addEventListener('click', () => {
          if (toolIsOn) {
            addAnnotation()
          }
        })
      })
    }

    assignEventListenersToPagesArray()

    const addAnnotation = () => {
      let widget
      let formField
      const fieldName = `form-field-${ Math.random() }`
      const today = new Date()

      switch (true) {
        case widgetType === 'text-field':
          widget = new PSPDFKit.Annotations.WidgetAnnotation({
            id: PSPDFKit.generateInstantId(),
            pageIndex: elIndex,
            formFieldName: fieldName,
            boundingBox: new PSPDFKit.Geometry.Rect({
              left: annoPosition.left,
              top: annoPosition.top,
              width: widgetWidth,
              height: widgetHeight
            }),
            font: 'Helvetica',
            fontColor: PSPDFKit.Color.BLACK,
            fontSize: 12,
            horizontalAlign: 'left',
            borderColor: PSPDFKit.Color.BLACK,
            borderWidth: 0,
            borderStyle: 'solid',
            createdAt: today,
            updatedAt: today
          })

          formField = new PSPDFKit.FormFields.TextFormField({
            name: fieldName,
            annotationIds: new (PSPDFKit.Immutable.List)([widget.id])
          })

          break
      }

      instance.create([widget, formField])

      pagesArray.forEach((page) => {
        page.classList.remove('create-tool')

        if (page.contains(cursor)) {
          page.removeChild(cursor)
        }
      })

      toolIsOn = false
    }

    toolbar?.addEventListener('click', () => {
      if (toolIsOn) {
        pagesArray.forEach((page) => {
          page.classList.remove('create-tool')
        })

        toolIsOn = false
      }
    })

    const clickOnFormTool = (width, height, type) => {
      toolIsOn = true

      pagesArray.forEach((page) => {
        page.classList.add('create-tool')
      })

      console.log(document.querySelector('.tool-cursor'))

      widgetWidth = width
      widgetHeight = height
      widgetType = type
    }

    const toolbarItems = [
      {
        type: 'custom',
        title: 'Text field',
        id: 'text-field',
        dropdownGroup: 'form-tools',
        icon: '<svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="currentColor" viewBox="0 0 24 24" focusable="false"><path d="M5 8.25c.41 0 .75.34.75.75v6c0 .41-.34.75-.75.75s-.75-.34-.75-.75V9c0-.41.34-.75.75-.75Zm9.21 2.25h5.59c.45 0 .67.54.35.85l-2.79 2.79c-.2.2-.51.2-.71 0l-2.79-2.79c-.32-.31-.09-.85.35-.85ZM22 5H2C.9 5 0 5.9 0 7v10c0 1.1.9 2 2 2h20c1.1 0 2-.9 2-2V7c0-1.1-.9-2-2-2Zm.5 12c0 .28-.22.5-.5.5H2c-.28 0-.5-.22-.5-.5V7c0-.28.22-.5.5-.5h20c.28 0 .5.22.5.5v10Z"></path></svg>',
        onPress: () => {
          clickOnFormTool(125, 30, 'text-field')
        }
      }
    ]

    instance.setToolbarItems(() => toolbarItems)
  })
})

onBeforeUnmount(() => {
  PSPDFKit.unload('.pdf-container')
})
</script>

<template>
  <div class="pdf-container"></div>
</template>

<style>
.pdf-container {
  width: calc(100vw - 100px);
  height: 100vh;
  margin: auto;
}
</style>