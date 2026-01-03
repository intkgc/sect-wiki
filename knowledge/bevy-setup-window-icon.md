---
title: Setup window icon in bevy 0.17
---

I ran into an issue where Bevy doesn’t provide an easy way to set an application icon. I found an online guide that explains how to do this for version 0.13, but that guide is outdated and no longer maintained. I took the code from it, adapted it for Bevy 0.17, and it works correctly.

Here is the code:

```rust
use bevy::ecs::system::NonSendMarker;
use bevy::prelude::*;
use bevy_winit::WINIT_WINDOWS;
use winit::window::Icon;

// Sturtup system
pub fn set_window_icon(_marker: NonSendMarker) {
    WINIT_WINDOWS.with_borrow(|winit_windows| {
        if winit_windows.windows.len() == 0 {
            return;
        }

        let (icon_rgba, icon_width, icon_height) = {
            let image = image::open("path/to/icon.png")
                .expect("Failed to open icon path")
                .into_rgba8();
            let (width, height) = image.dimensions();
            let rgba = image.into_raw();
            (rgba, width, height)
        };

        let icon = Icon::from_rgba(icon_rgba, icon_width, icon_height).unwrap();

        for window in winit_windows.windows.values() {
            window.set_window_icon(Some(icon.clone()));
        }

        info!("Window icon set");
    });
}
```

Note: you need to add `winit` and `image` to your project’s dependencies, and they must be the same versions as used by Bevy. As of Bevy 0.17, that should be `winit = "0.30.12"` and `image = "0.25.9"`. If you don’t know which version to use, you can use `cargo tree` or check `Cargo.lock` to see which is the correct version.
