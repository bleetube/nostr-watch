<template>
  <span  
      v-if="this.store.tasks.getActiveSlug === taskSlug && isLoggedIn"
      class="text-white lg:text-sm mr-10 ml-2 mt-1.5 text-xs">
    <span>Retrieving Relays List...</span>
  </span>
</template>

<style scoped>

</style>

<script>
import crypto from 'crypto'
// import { RelayPool } from 'nostr'

import { defineComponent, toRefs } from 'vue'

import { setupStore } from '@/store'

import SharedMethods from '@/shared/relays-lib.js'
import UserMethods from '@/shared/user-lib.js'
import SharedComputed from '@/shared/computed.js'

import { relays } from '../../../../relays.yaml'

const localMethods = new Object()

localMethods.invalidate = function(force){
  if( !this.isExpired(this.taskSlug) && !force ) 
    return
  
  if( !this.isLoggedIn() ) 
    return

  if( !this.store.prefs.useKind3 )
    return
  
  // console.log('wtf?', this.taskSlug, !this.isExpired(this.taskSlug) && !force)

  this.queueKind3(this.taskSlug)
}

localMethods.timeUntilRefresh = function(){
  return this.timeSince(Date.now()-(this.store.tasks.getLastUpdate(this.taskSlug)+this.store.prefs.duration-Date.now())) 
}

localMethods.timeSinceRefresh = function(){
  return this.timeSince(this.store.tasks.getLastUpdate(this.taskSlug)) || Date.now()      
}

localMethods.hash = function(relay){
  return crypto.createHash('md5').update(relay).digest('hex');
}

export default defineComponent({
  name: 'TemplateTask',
  components: {},
  data() {
    return {
      taskSlug: 'user/relay/list',
      kind3Remote: new Object(),
      kind3Local: {}
    }
  },
  setup(props){
    const {resultsProp: results} = toRefs(props)
    const {forceProp: force} = toRefs(props)
    return { 
      store : setupStore(),
      results: results,
      force: force
    }
  },
  created(){
    clearInterval(this.interval)
  },
  unmounted(){
    clearInterval(this.interval)
  },
  beforeMount(){
    this.lastUpdate = this.store.tasks.getLastUpdate(this.taskSlug)
    this.untilNext = this.timeUntilRefresh()
    this.sinceLast = this.timeSinceRefresh()
    
    this.relays = Array.from(new Set(relays))
  },
  mounted(){
    // console.log('task', this.taskSlug, 'is processing:', this.store.tasks.isProcessing(this.taskSlug))
    if(this.store.tasks.isProcessing(this.taskSlug))
      this.invalidate(true)
    else
      this.invalidate(this.force)
  },
  updated(){},
  computed: Object.assign(SharedComputed, {
    getDynamicTimeout: function(){
      return this.averageLatency*this.relays.length
    },
  }),
  methods: Object.assign(localMethods, UserMethods, SharedMethods),
  props: {
    resultsProp: {
      type: Object,
      default(){
        return {}
      }
    },
    forceProp: {
      type: Boolean,
      default(){
        return false
      }
    },
  },
})
</script>

<style scoped>
 #refresh { font-size: 12pt; color:#666; margin-bottom:15px }
 #refresh button { cursor: pointer; border-radius: 3px; border: 1px solid #a0a0a0; color:#333 }
 #refresh button:hover {color:#000;}
 #refresh button[disabled] {color:#999 !important; border-color:#e0e0e0}
</style>

