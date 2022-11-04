# rnbo-svelte

*rnbo-svelte* is a node package for abstracting away repetitive tasks that are needed to work with [RNBO](https://www.npmjs.com/package/@rnbo/js) in a node-based application.

## What does it offer?

As of writing this, there are three major things offered by this package.

`createDeviceInstance(devicePath, context, output)` is a function for creating a new instance of an RNBO device. You `devicePath` is a `JSON` bundle exported from RNBO and you also supply an AudioContext object and an output GainNode.

`sendDeviceMessage(device, tag, message)` is a function for sending a message to an RNBO device via an `inport`. I tend to prefer the `inport` instead of the `param` as it allows you to send arrays into the device. You lose the ability to setup `@enum` and set minima and maxima, but that's a worthwhile tradeoff in my personal use cases.

`loadSamples` is a function for easily loading a bundle of samples into buffers which are structured as a `JSON` object. Because this returns a promise, it supports chaining it after a device instance is created, ideally via `createDeviceInstance`.

## Usage Examples

You can see an example [here](https://github.com/intersymmetric/intersymmetric/blob/643dd20aee8ff6f3df2c936469940ec192ecf1c9/src/lib/nyege/Instance.svelte#L1) in the [intersymmetric project](https://intersymmetric.xyz/nnnb) of `loadSamples` and `createDeviceInstance` being used to instantiate a device and then load a bundle of audio samples into the devices buffers.

```sv
<script>
    import Interface from './Interface.svelte';
    import Button from './Button.svelte';
    import { loadSamples, createDeviceInstance } from '@jamesb93/rnbo-svelte';

    let patch, context;
    let samplesLoaded = false;

    const start = async () => {
        context = new (window.AudioContext || window.webkitAudioContext)();
        let output = context.createGain().connect(context.destination);
        createDeviceInstance('/nyege/code/patch.export.json', context, output)
        .then(response => {
            patch = response;
            const samples = new Array(33).fill(0).map((_, i) => ({
                url: `/nyege/samples/${i}.mp3`,
                buffer: `b.${i+1}`
            }))
            loadSamples(patch, context, samples);
            samplesLoaded = true;
        });

    };
</script>
```

You can also see an example of `sendDeviceMessage` [here](https://github.com/intersymmetric/intersymmetric/blob/643dd20aee8ff6f3df2c936469940ec192ecf1c9/src/lib/nyege/interface/A/Module.svelte#L72), where a reactive statement fires off messages to the device everytime some state in a Svelte store changes.

```sv
$: sendDeviceMessage(patch, 'polymetric_params', [0, $range0, $div0]);
```

## Contribution

Open a PR, or just get in touch with me at github@jamesbradbury.net.

