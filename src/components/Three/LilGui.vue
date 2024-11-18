<script setup lang="ts">
  import { GUI } from 'lil-gui'
  import { onMounted, ref } from 'vue'

  const guiRef = ref()

  onMounted(() => {
    const gui = new GUI({ container: guiRef.value })

    let preset = {}

    const obj = {
      myBoolean: true,
      myString: 'lil-gui',
      myNumber: 1,
      size: 'small',
      color: { r: 0, g: 0.2, b: 0.4 },
      x: 0,
      y: 0,
      z: 0,
      myFunction: () => {
        alert('hi')
      },
      savePreset() {
        preset = gui.save()
        loadButton.enable()
      },
      loadPreset() {
        gui.load(preset)
      },
    }

    gui.add(document, 'title')

    gui.add(obj, 'myBoolean')
    gui.add(obj, 'myString')
    gui.add(obj, 'myNumber', 0, 100, 5)
    gui.add(obj, 'myFunction')
    gui.add(obj, 'size', ['Small', 'Medium', 'Large'])
    gui.addColor(obj, 'color')

    const folder = gui.addFolder('Position')
    folder.add(obj, 'x', 0, 100, 1).onChange((e: any) => {
      obj.y = obj.x * 2
      console.log('111', e, obj.y)
    })
    folder.add(obj, 'y', 0, 100, 1).listen()
    folder.add(obj, 'z', 0, 100, 1)
    gui.add(obj, 'savePreset')
    const loadButton = gui.add(obj, 'loadPreset')
    loadButton.disable()
  })
</script>

<template>
  <div id="gui" ref="guiRef"></div>
</template>

<style scoped lang="scss">
  #gui {
    position: fixed;
    top: 10px;
    right: 10px;
  }
  ::v-deep(.lil-gui) {
    color: #fff;
    // --width: 320px;
    --widget-color: #313131;
    --padding: 8px;
  }
</style>
