# Santiago under the lid

**Live page: https://likrentai.github.io/santiagosmog/**

**Case study: https://likrentai.github.io/santiagosmog/case-study.html** ([PDF](https://likrentai.github.io/santiagosmog/case-study.pdf))

For five days in June 2026, Santiago de Chile lived under smog alerts. This page rebuilds the city in 3D, about 72,000 buildings on the real terrain of the basin with the Andes behind, and replays the episode from 11 to 19 June 2026 hour by hour.

- **The smog** is drawn from the 24-hour PM2.5 average at nine SINCA monitoring stations, spread between them, and from the lid: the height of the boundary layer in ERA5. The peak was 132 µg/m³ at Pudahuel and Cerro Navia on 14 June.
- **The lid** sinks to 13 to 30 m every night of the episode and reached only 140 to 460 m in the afternoons, until 685 m on 18 June ended the episode.
- **The page** has a time bar (1 to 8 hours a second), an info box with the declared level (alerta or preemergencia), an episode chart you can click to jump to any hour, station pins with their readings, and a short explainer of how the lid traps smog. The camera is yours: nothing turns on its own.

The page needs a desktop or laptop with a graphics card. It loads about 12 MB of building data.

## How it is made

Five notebooks in Python 3.14 build the data. The page itself is plain HTML with deck.gl (its TerrainLayer draws the ground), and it needs no build step.

| Notebook | What it does |
|---|---|
| `00buildings.ipynb` | Downloads the OpenStreetMap extract for Chile from Geofabrik (352 MB) and keeps the buildings in a 16 by 9 km box around central Santiago (`data/buildings_santiago.parquet`). |
| `01webbuildings.ipynb` | Gives every building a height: mapped in OpenStreetMap (21%), by building type (38%), or from the GHS-BUILT-H 100 m grid (41%). Reads each building's ground level from the Terrarium elevation tiles and writes `docs/data/buildings.json` (71,773 buildings). |
| `02smogdata.ipynb` | Downloads PM2.5 for the SINCA stations from the OpenAQ archive, and the boundary layer height, wind and temperature from ERA5 through Open-Meteo, for nine points across the basin. |
| `03websmog.ipynb` | Removes a one-hour glitch inside the 24-hour averages, builds the city lid and an illustrative residual layer, and writes `docs/data/smog.json`. |
| `04casestudy.ipynb` | Draws the episode chart for the case study (`docs/images/cs_episode.png`). |

To rebuild, create an environment and install the packages in `requirements.txt`. Then run the notebooks in order from the repository folder. To view the page, run `python -m http.server 8000` inside `docs` and open http://localhost:8000.

## Data sources

- **PM2.5** (24-hour rolling average, hourly): SINCA, Ministerio del Medio Ambiente de Chile, through the [OpenAQ](https://openaq.org) archive. Stations: Talagante, Pudahuel, Cerro Navia, Quilicura, El Bosque, Parque O'Higgins, Puente Alto, La Florida and Las Condes.
- **Boundary layer height, wind at 10 m and temperature at 2 m**: ERA5 reanalysis (Copernicus Climate Change Service, ECMWF), through the [Open-Meteo](https://open-meteo.com) historical weather API, CC BY 4.0.
- **Buildings**: (c) OpenStreetMap contributors, ODbL, from the [Geofabrik](https://download.geofabrik.de) extract for Chile.
- **Building heights where OpenStreetMap has none**: GHS-BUILT-H R2023A, average net building height 2018, European Commission Joint Research Centre.
- **Elevation**: Mapzen Terrarium tiles on AWS open data.
- **Imagery and hillshade**: Esri World Imagery (Esri, Maxar, Earthstar Geographics) and Esri World Hillshade, loaded by the page.
- **Declared episodes**: news reports (La Tercera, Emol, BioBioChile, T13). Preemergencia on 13, 14, 15 and 17 June; alerta on 16 June.

## Limits

- PM2.5 is the 24-hour rolling average, as archived by OpenAQ, not hourly readings. It is also the measure used to declare episodes. The rhythm within each day comes from the lid.
- PM2.5 between the nine stations is spread by inverse distance weighting, so local hot spots are missed.
- The lid is one ERA5 value for the whole city (two ERA5 cells cover Santiago). At night it is the depth of air that still mixes, not the top of the inversion; the residual layer above it is an illustration.
- The cloud texture, the puffs, the drift and the snow above 1,500 m are illustrations. The smog does not move with the wind.
- Smog density follows the square of PM2.5 above 15 µg/m³, a visual choice so that moderate days look much clearer than the worst ones. The haze in the distance assumes about 3 km visibility at 130 µg/m³.
- 79% of the building heights are estimated. The Gran Torre Costanera is modelled from its published shape.

## Repository layout

```
00buildings.ipynb ... 04casestudy.ipynb    data notebooks
docs/index.html                            the page
docs/data/                                 data used by the page
docs/case-study.html, case-study.pdf       the case study
docs/images/, docs/fonts/                  case study pictures and fonts
requirements.txt                           Python packages for the notebooks
```

`data/` (the OpenStreetMap extract, the GHS-BUILT-H tile, the elevation tiles and the downloaded PM2.5 and ERA5 files) is not in the repository. The notebooks download or rebuild it.

## Licence

The code is under the MIT licence (see `LICENSE`). The data files in `docs/data` are derived from the sources above and remain under their licences. Please credit SINCA, OpenAQ, Copernicus (ERA5), Open-Meteo, OpenStreetMap contributors and the European Commission JRC when you reuse them.
