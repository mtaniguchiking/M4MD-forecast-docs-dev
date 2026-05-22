---
title: Folder structure
weight: 3
bookToc: true
---

# Folder structure

A best practice is to create a M4MD_NPS folder in the same directory but separate from the M4MD folder (i.e., have C:\M4MD and C:\M4MD_NPS). This helps ensure that you do not inadvertently push sensitive information to GitHub. Within your M4MD_NPS folder, you should setup the file tree structure for your given network (`<network code>`) and park units (`<unit code>`) to look like the following:

    . # C:\M4MD_NPS (outside M4MD project folder)
    └── <config>/
        └── <network code>/
            ├── <unit code1>/
            │   ├── _park-level-attributes.yml
            │   └── <analysis file>.yml
            ├── <unit code2>/
            │   ├── _park-level-attributes.yml
            │   └── <analysis file>.yml
            └── _network-level-attributes.yml
    └── <data>/
        └── <network code>/
            ├── <unit code1>/
            ├── <unit code2>/
    └── <output>/
        └── <network code>/
            ├── <unit code1>/
            ├── <unit code2>/
         
The required network-level-attributes.yml and park-level-attributes.yml files are described in  [Guide/Config files/Design metadata]({{< ref "/docs/1-guide/b-config-files/i-design-metadata.md" >}}).

To take a real-world example from an analysis of species richness data at Little Bighorn Battlefield National Monument (LIBI), in Montana -- a park within Rocky Mountain Network (ROMN) -- the complete file tree might look like the following:

    .
    └── M4MD_NPS
        └── config/
            └── ROMN/
                ├── LIBI/
                │   ├── _park-level-attributes.yml
                │   └── richness.yml
                └── _network-level-attributes.yml
          └── <data>/
            └── <ROMN>/
                ├── <LIBI>/
                    ├── LIBI_richness.csv
                    ├── LIBI_sites.csv
                    ├── LIBI_covars.csv
        └── <output>/
            └── <ROMN>/
                ├── <LIBI>/
                    ├── <model>/

