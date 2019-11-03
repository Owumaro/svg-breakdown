<template>
  <div>
    <!-- Navbar -->
    <nav class="navbar navbar-light border-bottom">
      <span class="navbar-brand">SVG breakdown</span>
      <a href="https://github.com/Owumaro" target="_blank">
        <img src="https://avatars2.githubusercontent.com/u/5205295?s=150" width="40" height="40" alt="">
      </a>
    </nav>

    <main class="main-container">
      <div class="sidebar border-right">
        <!-- File -->
        <div class="p-3">
          <!-- File input -->
          <div class="form-group">
            <label for="svgFileInput">SVG file</label>
            <b-form-file
              id="svgFileInput"
              accept="image/svg+xml"
              v-model="svgFile" />
          </div>

          <!-- File display -->
          <div class="bg-checkerboard" v-html="svgFileContent"></div>
        </div>

        <!-- Controls -->
        <div v-if="parsed" class="p-3 border-top">
          <!-- Buttons -->
          <div class="d-flex gutter-sm justify-content-center align-items-center mb-2">
            <button
              type="button"
              class="btn btn-sm btn-primary rounded-circle"
              @mousedown="previousStepStart"
              @mouseup="previousStepEnd"
              @mouseleave="previousStepEnd">
              <svg aria-hidden="true" focusable="false" data-prefix="fas" data-icon="step-backward" role="img" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 448 512" class="fa-icon"><path fill="currentColor" d="M64 468V44c0-6.6 5.4-12 12-12h48c6.6 0 12 5.4 12 12v176.4l195.5-181C352.1 22.3 384 36.6 384 64v384c0 27.4-31.9 41.7-52.5 24.6L136 292.7V468c0 6.6-5.4 12-12 12H76c-6.6 0-12-5.4-12-12z" class=""></path></svg>
            </button>
            <div>{{ currentStep }} / {{ parsed.totalSteps }}</div>
            <button
              type="button"
              class="btn btn-sm btn-primary rounded-circle"
              @mousedown="nextStepStart"
              @mouseup="nextStepEnd"
              @mouseleave="nextStepEnd">
              <svg aria-hidden="true" focusable="false" data-prefix="fas" data-icon="step-forward" role="img" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 448 512" class="fa-icon"><path fill="currentColor" d="M384 44v424c0 6.6-5.4 12-12 12h-48c-6.6 0-12-5.4-12-12V291.6l-195.5 181C95.9 489.7 64 475.4 64 448V64c0-27.4 31.9-41.7 52.5-24.6L312 219.3V44c0-6.6 5.4-12 12-12h48c6.6 0 12 5.4 12 12z" class=""></path></svg>
            </button>
          </div>

          <!-- Progress bar -->
          <div class="progress">
            <div
              class="progress-bar"
              role="progressbar"
              :style="`width: ${stepsPercentage * 100}%`"
              :aria-valuenow="currentStep"
              aria-valuemin="0"
              :aria-valuemax="parsed.totalSteps">
            </div>
          </div>
        </div>
      </div>

      <div class="content">
        <template v-if="!parsed">
          <div class="p-3">
            Please import a SVG file
          </div>
        </template>
        <template v-else>
          <div class="p-3 border-bottom panel-partial-svg-display">
            <div class="bg-checkerboard d-inline-block h-100" v-html="parsed.partialSvgFile"></div>
          </div>
          <div class="panel-partial-svg-content">
            <pre class="h-100 p-3 mb-0">{{ parsed.partialSvgFile }}</pre>
          </div>
        </template>
      </div>
    </main>
  </div>
</template>

<script>
import { BFormFile } from 'bootstrap-vue'
import format from 'xml-formatter'

export default {
  components: { BFormFile },
  data: function() {
    return {
      svgFile: null,
      svgFileContent: null,
      currentStep: 0,
      nextStepInterval: null,
      previousStepInterval: null,
      speed: 100
    }
  },
  computed: {
    // Parse svg file content to extract steps and compute partial svg file based on current step
    // Heavily inspired by https://gist.github.com/johan/1007786
    parsed: function() {
      if (this.svgFileContent) {
        const domParser = new DOMParser()
        const svgDocument = domParser.parseFromString(this.svgFileContent, 'image/svg+xml')
        const walker = document.createTreeWalker(svgDocument)

        let stepsApplied = 0
        let totalSteps = 0

        // Explore svg looking for paths
        let current
        while (current = walker.nextNode()) {
          if (current.nodeType === 1) { // ELEMENT_NODE
            const d = current.getAttribute('d')
            if (d) {
              // Found a node with a path, now split path and build partial svg
              const segs = d.split(/(?=[a-z])/i)
              totalSteps += segs.length

              if (stepsApplied >= this.currentStep) {
                // Mark for removal after walker loop
                //current.remove()
                current.setAttribute('toDelete', true)
                stepsApplied += segs.length
              } else if (stepsApplied + segs.length > this.currentStep) {
                // Create new path until currentStep is reached
                let newD = ''
                segs.forEach(seg => {
                  if (stepsApplied < this.currentStep) newD += seg
                  stepsApplied += 1
                })

                current.setAttribute('d', newD)
              } else {
                // Keep full path
                stepsApplied += segs.length
              }
            }
          }
        }

        // Remove paths marked to delete
        svgDocument.querySelectorAll('[toDelete=true]').forEach(node => {
          node.remove()
        })

        const partialSvgFile = format(new XMLSerializer().serializeToString(svgDocument), { collapseContent: true, indentation: '  ' })

        return {
          totalSteps,
          partialSvgFile
        }
      }
    },
    // Steps percentage
    stepsPercentage: function() {
      return this.parsed ? this.currentStep / this.parsed.totalSteps : null
    }
  },
  methods: {
    nextStepStart: function() {
      if (!this.nextStepInterval) {
        this.nextStepInterval = setInterval(() => {
          if (this.currentStep < this.parsed.totalSteps) this.currentStep += 1
        }, this.speed)
      }
    },
    nextStepEnd: function() {
      clearInterval(this.nextStepInterval)
      this.nextStepInterval = null
    },
    previousStepStart: function() {
      if (!this.previousStepInterval) {
        this.previousStepInterval = setInterval(() => {
          if (this.currentStep > 0) this.currentStep -= 1
        }, this.speed)
      }
    },
    previousStepEnd: function() {
      clearInterval(this.previousStepInterval)
      this.previousStepInterval = null
    }
  },
  watch: {
    // Read svg file content
    svgFile: function() {
      const reader  = new FileReader()

      reader.onload = (e) => {
        this.svgFileContent = e.target.result
        this.currentStep = 0
      }

      reader.readAsText(this.svgFile)
    }
  },
  created: function() {
    // Bind arrow keys to previous/next step
    window.addEventListener('keydown', e => {
      if (e.keyCode == '37') {
        this.previousStepStart()
      } else if (e.keyCode == '39') {
        this.nextStepStart()
      }
    })
    window.addEventListener('keyup', e => {
      if (e.keyCode == '37') {
        this.previousStepEnd()
      } else if (e.keyCode == '39') {
        this.nextStepEnd()
      }
    })
  }
}
</script>

<style lang="scss">
body, #app, #app > div {
  height: 100vh;
}

.main-container {
  display: grid;
  grid-template-columns: 300px 1fr;
  grid-template-areas: "sidebar content";
}

.sidebar, .content {
  height: calc(100vh - 3.5rem - 1px);
  overflow: auto;
}

.bg-checkerboard {
  background-image:
    /* tint image */
    linear-gradient(to right, rgba(192, 192, 192, 0.75), rgba(192, 192, 192, 0.75)),
    /* checkered effect */
    linear-gradient(to right, black 50%, white 50%),
    linear-gradient(to bottom, black 50%, white 50%);
  background-blend-mode: normal, difference, normal;
  background-size: 1rem 1rem;
}

.panel-partial-svg-display {
  width: 100%;
  height: 50%;
  position: relative;
  text-align: center;
}

.panel-partial-svg-content {
  height: 50%;
}

svg {
  height: 100%;
  max-width: 100%;
}
</style>
