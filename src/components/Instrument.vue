<template lang="pug">
div
    .instrument
        .controls
            div
                label Steps: 
                input(type="text", v-model.number='steps')
            div
                label Measures: 
                input(type="text", v-model.number='measures')
            div
                label Transpose (semitones): 
                input(type="text", v-model.number='transpose')
            div
                label Velocity (%): 
                input(type="text", v-model.number='volume')
            div
                label Synth type: 
                select(v-model="synth")
                    option sine
                    option sawtooth
                    option square
                    option triangle
            div
                label Decay (seconds): 
                input(type="text", v-model.number='decay')
            div
                label Filter cutoff (hz): 
                input(type="text", v-model.number='filterCutoff')
            //div
                //label Reverb %: 
                //input(type="text", v-model.number='reverbWet')
        div.boxes
            line-c(:beats = "steps", :current-note="currentStep", v-for="(onNotes, pitch) in notes", :pitch = "pitch", :on-notes="onNotes", @toggle-box = "handleToggleBox")

    hr   
</template>

<script lang="coffee">

Hello = require './Hello'
LineC = require './Line'

Tone = require('tone')

module.exports = {
    name: 'instrument',
    components: {
        Hello, LineC
    },
    props: ['time', 'res']
    data: () ->
        return {
            notes : {
                "C5" : []
                "B4" : []
                "A4" : []
                "G4" : []
                "F4" : []
                "E4" : []
                "D4" : []
                "C4" : []
            },
            steps: 16
            measures: 4
            synth: "sine"
            transpose: 0
            filterCutoff: 2000
            # reverbWet: 10
            volume: 100
            decay: 2.0
        }
    computed: {
        currentStep: () ->
            ticksPerStep = this.measures * this.res / this.steps
            return Math.floor(this.time / ticksPerStep) % this.steps
    }
    watch: {
        currentStep: () ->
            this.playNextStep()
        steps: () ->
            for key of this.notes
                this.notes[key] = []
        synth: (val) ->
            this.player.set({
                oscillator: {
                    type: val
                }
            })
        transpose: (val) ->
            if isNaN(parseInt(val))
                val = 0
            this.player.set({
                oscillator: {
                    detune: val * 100
                }
            })
        filterCutoff: (val) ->
            if !isNaN(parseInt(val))
                this.filter.set({
                    frequency: val
                })
        # reverbWet: (val) ->
        #     if !isNaN(parseInt(val))
        #         this.reverb.set(wet: val / 100)            
        volume: (val) ->
            if !isNaN(parseInt(val))
                this.player.set(volume: val / 100) 
        decay: (val) ->
            if !isNaN(parseInt(val))
                this.player.set(envelope: {decay: val, release: val}) 

    }
    methods: {
        playNextStep: () ->
            self = this
            for pitch, onNotes of this.notes
                if (this.currentStep in onNotes)
                    this.player.triggerAttackRelease(pitch, "8n", "+0", this.volume / 100)

        handleToggleBox: (pitch, pos) ->
            if pos in this.notes[pitch]
                index = this.notes[pitch].indexOf(pos);
                if(index != -1)
                    this.notes[pitch].splice( index, 1 );
            else
                this.notes[pitch].push(pos)
                this.player.triggerAttackRelease(pitch, "8n", "+0", this.volume / 100)

    }

    created: () ->

        synth = new Tone.PolySynth(4, Tone.OmniSynth)

        synth.set({
            "oscillator" : {
                "type" : this.synth
                detune: this.transpose * 100
            }
        })

        defaultEnv = {
            "envelope" : {
                "attack" : 0.01,
                "decay" : this.decay,
                "sustain" : 0.0,
                "release" : this.decay,
            }
        }

        synth.set(defaultEnv)
        this.filter = new Tone.Filter(this.filterCutoff, "lowpass");

        this.synthPlayer = synth

        # this.reverb = new Tone.Freeverb()
        # this.reverb.set(wet: this.reverbWet / 100)

        synth.connect(this.filter)
        this.filter.connect(Tone.Master)
        # this.reverb.connect(Tone.Master)
        this.player = synth
        this.player.set(volume: this.volume / 100)

}
</script>

<style lang="stylus">
.instrument
    margin-bottom: 50px
    display:flex
    flex-direction: row

input, select
    margin: 10px
.controls
    width: 25%
    height: 100%
.boxes
    width: 75%
    
</style>
