---
updated: 2018-01-26 16:00
authors:
- name: Anil Madhavapeddy
  uri: http://anil.recoil.org
  email: anil@recoil.org
subject: MirageOS running on the ESP32 embedded chip
permalink: 2018-esp32-booting
---

Now that the winter holiday break is over, we are starting to see the results of winter hacking among our community.

The first great hack for 2018 is from [Sadiq Jaffer](http://toao.com), who got OCaml booting on a tiny and relatively new CPU architecture -- the [Espressif ESP32](http://esp32.net).  After proudly demonstrating a battery powered version to the folks at [OCaml Labs](https://ocamllabs.io), he then proceeded to clean it up enough tha it can be built with a [Dockerfile](https://github.com/sadiqj/ocaml-esp32-docker), so that others can start to do a native code port and get bindings to the networking interface working.

[Read all about it on Sadiq's blog](http://toao.com/blog/getting-ocaml-running-on-the-esp32#getting-ocaml-running-on-the-esp32), and thanks for sharing this with us, Sadiq!

We also noticed that another OCaml generic virtual machine for even smaller microcontrollers has [shown up on GitHub](https://github.com/stevenvar/omicrob).  This, combined with some functional metaprogramming, may mean that 2018 is the year of OCaml in all the tiny embedded things...

