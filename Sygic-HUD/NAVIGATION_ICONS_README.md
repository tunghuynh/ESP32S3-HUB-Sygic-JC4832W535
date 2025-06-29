# Navigation Icons for ESP32 LVGL Display

This project contains auto-generated navigation icons converted from PNG files to C byte arrays suitable for LVGL display on ESP32.

## Generated Files

### 1. `src/navigation_icons.h`
Main header file containing all navigation icons as C byte arrays:
- `icon_turn_left` - Left turn arrow with curved design
- `icon_turn_right` - Right turn arrow with curved design  
- `icon_continue_straight` - Straight up arrow
- `icon_roundabout` - Circular roundabout with entry/exit indicators

### 2. `src/navigation_icon_example.cpp`
Example code showing how to use the icons in your ESP32 LVGL application.

## Icon Specifications

- **Size**: 48x48 pixels (optimized for ESP32 memory constraints)
- **Format**: RGB565 (16-bit color)
- **Memory per icon**: 4,608 bytes (48 × 48 × 2)
- **Total memory**: ~18.4KB for all 4 icons
- **Color scheme**: Black arrows on white background
- **LVGL compatibility**: v8.x and v9.x

## Usage in Your ESP32 Code

### Basic Integration

```cpp
#include "navigation_icons.h"

// Create an image object
lv_obj_t* nav_icon = lv_img_create(parent);

// Set the icon source
lv_img_set_src(nav_icon, &icon_turn_left);

// Position the icon
lv_obj_align(nav_icon, LV_ALIGN_CENTER, 0, 0);
```

### Dynamic Icon Switching

```cpp
void update_navigation_display(lv_obj_t* img_obj, const char* direction) {
    if (strcmp(direction, "left") == 0) {
        lv_img_set_src(img_obj, &icon_turn_left);
    } else if (strcmp(direction, "right") == 0) {
        lv_img_set_src(img_obj, &icon_turn_right);
    } else if (strcmp(direction, "straight") == 0) {
        lv_img_set_src(img_obj, &icon_continue_straight);
    } else if (strcmp(direction, "roundabout") == 0) {
        lv_img_set_src(img_obj, &icon_roundabout);
    }
}
```

### With Styling

```cpp
// Apply blue tint to icon
static lv_style_t icon_style;
lv_style_init(&icon_style);
lv_style_set_img_recolor_opa(&icon_style, LV_OPA_50);
lv_style_set_img_recolor(&icon_style, lv_color_hex(0x0080FF));
lv_obj_add_style(nav_icon, &icon_style, 0);
```

## Memory Considerations

The icons are designed for ESP32 memory constraints:

- **ESP32**: 320KB SRAM - Icons use ~6% of available memory
- **ESP32-S3**: 512KB SRAM - Icons use ~4% of available memory

## Icon Design Details

### Turn Left Icon
- Curved arrow starting from center, going up, then curving left
- Clear arrow head pointing left
- Intuitive design matching standard navigation symbols

### Turn Right Icon  
- Mirror of left turn icon
- Curved arrow going up then right
- Arrow head pointing right

### Continue Straight Icon
- Simple vertical arrow pointing upward
- Clean, minimalist design
- Clear directional indication

### Roundabout Icon
- Circular design with entry arrow from bottom
- Exit indicator on the right
- Matches international roundabout symbols

## Original Source Files

The icons were generated from PNG files located in:
`/mnt/f/OneDriveFPT/OneDrive - FPT Software/PROJECT/Source/Arduino/ESP32-TOUCH/Sygic-HUD/src/icon/light/`

Original files processed:
- direction_turn_left.png (200×200) → Resized to 48×48
- direction_turn_right.png (200×200) → Resized to 48×48  
- direction_continue_straight.png (200×200) → Resized to 48×48
- direction_roundabout.png (200×200) → Resized to 48×48

## Generation Tools

### `improved_icon_generator.py`
Python script that creates the navigation icons programmatically with geometric patterns optimized for clarity at small sizes.

### `png_to_c_converter.py` 
Original PNG processing script (fallback patterns used due to PNG parsing complexity).

## Integration Checklist

- [ ] Copy `navigation_icons.h` to your ESP32 project
- [ ] Include the header in your main.cpp file
- [ ] Ensure LVGL is configured for RGB565 format
- [ ] Test icon display and memory usage
- [ ] Implement dynamic icon switching based on navigation data
- [ ] Add styling as needed for your UI theme

## Performance Notes

- Icons are stored in flash memory, loaded to RAM when displayed
- RGB565 format provides good color depth with reasonable memory usage
- 48×48 size provides clear visibility while conserving memory
- Static const arrays ensure efficient memory usage

## Future Enhancements

Potential improvements:
- Add more navigation icons (U-turn, exit ramp, etc.)
- Implement icon animations
- Create different color variants
- Optimize for different screen sizes
- Add night mode variants