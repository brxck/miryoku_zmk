# Miryoku ZMK - Corne nice_nano_v2 Customization

## Hardware
- Board: nice_nano_v2
- Shield: corne (3x6 split, 42 keys)
- Build workflow: `.github/workflows/build.yml`

## Architecture

Miryoku defines 10 layers with 40 logical keys each (3x5 per side + 3 thumbs per side). The Corne has 42 physical keys (3x6 + 3 thumbs). The 6 extra outer-column positions are where customization happens.

### Include chain
```
config/corne.keymap
  -> miryoku/custom_config.h        # OUR CUSTOMIZATIONS GO HERE
  -> miryoku/mapping/42/corne.h     # physical layout mapping (42 keys)
  -> miryoku/miryoku.dtsi           # keymap generation
       -> miryoku/miryoku.h
            -> miryoku_babel/miryoku_layer_selection.h  # per-layer mapping/content selection
            -> miryoku_babel/miryoku_layer_list.h       # layer names and indices
```

### How layers are generated
In `miryoku.dtsi`, each layer expands via:
```c
bindings = < U_MACRO_VA_ARGS(MIRYOKU_LAYERMAPPING_##LAYER, MIRYOKU_LAYER_##LAYER) >;
```
- `MIRYOKU_LAYER_<LAYER>` = the 40 key bindings for that layer
- `MIRYOKU_LAYERMAPPING_<LAYER>` = the mapping macro that arranges 40 logical keys into the physical layout

Each `MIRYOKU_LAYERMAPPING_<LAYER>` defaults to `MIRYOKU_MAPPING` (= `MIRYOKU_LAYOUTMAPPING_CORNE`) which maps outer columns to `&none`. Defining `MIRYOKU_LAYERMAPPING_<LAYER>` in `custom_config.h` overrides this per-layer, because `miryoku_layer_selection.h` guards each with `#if !defined(...)`.

### Layers
BASE, EXTRA, TAP, BUTTON, NAV, MOUSE, MEDIA, NUM, SYM, FUN (indices 0-9)

## Current Customizations (`miryoku/custom_config.h`)

`MIRYOKU_LAYERMAPPING_BASE` overrides the BASE layer's physical mapping to place bindings on the 6 outer-column keys:

```
Left outer column (top to bottom):
  Alt+Shift+Grave          &kp LA(LS(GRAVE))
  Hyper+X                  &kp LG(LS(LA(LC(X))))
  Hyper+Y                  &kp LG(LS(LA(LC(Y))))

Right outer column (top to bottom):
  Hyper+C                  &kp LG(LS(LA(LC(C))))
  Hyper+D                  &kp LG(LS(LA(LC(D))))
  Hyper+E                  &kp LG(LS(LA(LC(E))))
```

Other layers use the default mapping (outer columns = `&none`).

## How to Customize

### Change outer column keys on BASE layer
Edit the bindings in `miryoku/custom_config.h` directly. Replace `&kp LG(LS(LA(LC(X))))` etc. with any ZMK binding.

### Add outer column keys to another layer
Add a new `MIRYOKU_LAYERMAPPING_<LAYER>` macro to `custom_config.h`. Copy the BASE macro and change the layer name and outer column bindings:
```c
#define MIRYOKU_LAYERMAPPING_NAV( \
     K00, K01, K02, K03, K04,      K05, K06, K07, K08, K09, \
     K10, K11, K12, K13, K14,      K15, K16, K17, K18, K19, \
     K20, K21, K22, K23, K24,      K25, K26, K27, K28, K29, \
     N30, N31, K32, K33, K34,      K35, K36, K37, N38, N39 \
) \
&kp SOMETHING    K00  K01  K02  K03  K04       K05  K06  K07  K08  K09  &kp OTHER \
&none            K10  K11  K12  K13  K14       K15  K16  K17  K18  K19  &none \
&none            K20  K21  K22  K23  K24       K25  K26  K27  K28  K29  &none \
                                K32  K33  K34       K35  K36  K37
```

### ZMK modifier syntax
- `LS()` = Left Shift, `RS()` = Right Shift
- `LC()` = Left Ctrl, `LA()` = Left Alt, `LG()` = Left GUI (Cmd)
- Nest for combos: `LG(LS(LA(LC(X))))` = Cmd+Shift+Alt+Ctrl+X (Hyper+X)
- Full reference: https://zmk.dev/docs/keymaps/list-of-keycodes

### Other Miryoku configuration defines
Set these in `custom_config.h` before includes:
- `MIRYOKU_ALPHAS_QWERTY` / `MIRYOKU_ALPHAS_COLEMAKDH` etc. — alpha layout
- `MIRYOKU_LAYERS_FLIP` — swap left/right layer placement
- `MIRYOKU_CLIPBOARD_MAC` — use Cmd-based clipboard keys
- `MIRYOKU_NAV_VI` — vi-style navigation
- `MIRYOKU_KLUDGE_THUMBCOMBOS` / `MIRYOKU_KLUDGE_TOPROWCOMBOS` — enable combo features

### Building
Trigger `.github/workflows/build.yml` via workflow_dispatch. Downloads firmware artifacts for both halves.

## Key Files
- `miryoku/custom_config.h` — all user customizations
- `miryoku/mapping/42/corne.h` — default Corne physical layout mapping (do not edit)
- `miryoku/miryoku_babel/miryoku_layer_alternatives.h` — all layer key definitions
- `miryoku/miryoku_babel/miryoku_layer_selection.h` — layer/mapping selection logic
- `miryoku/miryoku.dtsi` — keymap generation template
- `config/corne.keymap` — top-level keymap entry point
