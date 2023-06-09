<script>
  import { getContext, createEventDispatcher, onDestroy } from "svelte"
  import { Slider } from "@budibase/bbui" // Import Budibase components

  export let value
  export let label
  export let disabled
  export let field
  export let defaultValue
  export let min
  export let max
  export let step 
  export let showLabel
  export let validation
  export let error

  let fieldApi;
  let fieldState; 

  const dispatch = createEventDispatcher()

  const labelPos = getContext("field-group")?.labelPosition || "above"
  const { styleable, builderStore } = getContext("sdk")
  const component = getContext("component");
  const formContext = getContext("form");
  const formStepContext = getContext("form-step"); 
  
  const formApi = formContext?.formApi;
  $: formStep = formStepContext ? $formStepContext || 1 : 1;
  $: formField = formApi?.registerField(field, "number", 0, false, validation, formStep);

  $: value = parseFloat(fieldState?.value || defaultValue || min)
  //$: showLabel = labelPos === "above" ? false : true
  //$: if ($formField) setBoundariesFromFieldSchema($formField.fieldSchema)
  $: disabled = fieldState?.disabled || disabled || false

  $: unsubscribe = formField?.subscribe((value) => {
    fieldState = value?.fieldState;
    fieldApi = value?.fieldApi;
  });

  //$: console.log("min", min, "max", max, "step", step, "value", value, "label", label, "disabled", disabled, "field", field, "defaultValue", defaultValue, "labelPos", labelPos, "showLabel", showLabel)

  // This function will check the field Schema for min / max constraints and automatically set the props
  // If no constraints are detected, it will fall back to defaults
  /*
  function setBoundariesFromFieldSchema ( fieldSchema ) {
    let _min = fieldSchema.constraints.numericality.greaterThanOrEqualTo || 0
    let _max = fieldSchema.constraints.numericality.lessThanOrEqualTo || 100
    let _label = fieldSchema.name || "Slider Input"

    builderStore.actions.updateProp("min", _min)
    builderStore.actions.updateProp("max", _max)
    builderStore.actions.updateProp("label", _label)
  }
  */

  function handleChange( event ) {
    dispatch("change", event.detail)
    let newValue = event.detail
    if ( value !== newValue ) {
      value = newValue
      fieldApi.setValue(value)
    }
  }

  onDestroy(() => {
    fieldApi?.deregister();
    unsubscribe?.();
  });
</script>

{#if !formContext}
  <div class="placeholder">Form components need to be wrapped in a form</div>
{:else}
  <div use:styleable={$component.styles}>
    <div class="spectrum-Slider-labelContainer">
      <label class="spectrum-Slider-label" id="spectrum-Slider-label-8" for="spectrum-Slider-input-8">
          {#if showLabel}
              {label}
          {/if}                
      </label>
      <div class="spectrum-Slider-value" role="textbox" aria-readonly="true" aria-labelledby="spectrum-Slider-label-8">{value}</div>
    </div>
    {#if showLabel}
      <Slider id="spectrum-Slider-input-8" error={error} disabled={disabled} min={parseFloat(min)} max={parseFloat(max)} step={parseFloat(step)} value={value} on:change={handleChange}/>
    {:else}
      <Slider id="spectrum-Slider-input-8" label={label} labelPosition={labelPos} error={error} disabled={disabled} min={parseFloat(min)} max={parseFloat(max)} step={parseFloat(step)} value={value} on:change={handleChange}/>
    {/if}
  </div>
{/if}


