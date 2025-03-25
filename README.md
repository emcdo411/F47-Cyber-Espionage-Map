# F47-Cyber-Espionage-Map

## Overview

This repository contains the code and interactive map for the case study *"Predictive Analysis of F-47 NGAD Technology Safety from Espionage."* The project focuses on visualizing cyber theft hotspots in China targeting the F-47 Next Generation Air Dominance (NGAD) fighter technology in 2025. Using R and the `leaflet` package, we created an interactive map to highlight key cities involved in cyber espionage, with data on successful attacks and a detailed description for context.

The map is saved as an HTML file (`china_cyber_hotspots.html`) and can be viewed in any web browser. The code below includes all attempts to visualize the data, including earlier approaches using `ggmap` and the `maps` package, which encountered rendering issues, and the final successful approach using `leaflet`.

---

## Interactive Map

The interactive map is available as `china_cyber_hotspots.html`. To view it:

1. Download or clone this repository.
2. Open `china_cyber_hotspots.html` in a web browser (e.g., Chrome, Firefox).
3. Interact with the map:
   - Zoom and pan to explore China.
   - Click on markers to see the city name and number of successful cyber attacks (e.g., "Beijing<br>Successful Attacks: 120").

### Map Features
- **Basemap**: Stadia Maps ("alidade_smooth" style) with proper attribution.
- **Markers**: Circle markers for Beijing, Shanghai, Shenzhen, Guangzhou, and Chengdu, sized by the number of successful cyber attacks.
- **Popups**: Display city names and attack counts when clicked.
- **Description**: Includes a title and description in the top-right corner:
  - **Title**: "Hotspots of Cyber Theft in China Targeting F-47 Technology"
  - **Description**: "Map showing key cities in China with high cyber theft activity targeting F-47 NGAD technology (2025). Intensity reflects the number of successful cyber attacks, with Beijing, Shanghai, and Shenzhen as primary sources of espionage risk."

---

## Code

Below is the complete code used in this case study, including all approaches attempted to create the map.

### Initial Attempt: Using `ggmap` with Stadia Maps
This approach used `ggmap` to create a static 2D map with a heatmap, glowing points, and labels. However, it encountered persistent rendering issues (`Viewport has zero dimension(s)` error) in RStudio.

```R
# Load libraries
library(tidyverse)
library(ggmap)
library(viridis)
library(plotly)
library(ggrepel)

# Register Stadia Maps API key
register_stadiamaps("9c644007-0572-4892-915a-8da356fe40ae")

# Dataset for cyber theft origins in China
china_cyber_data <- tibble(
  City = c("Beijing", "Shanghai", "Shenzhen", "Guangzhou", "Chengdu"),
  Successful_Attacks = c(120, 90, 80, 50, 40),
  Latitude = c(39.9042, 31.2304, 22.5431, 23.1291, 30.5728),
  Longitude = c(116.4074, 121.4737, 114.0579, 113.2644, 104.0668)
)

# Get a map of China using get_stadiamap
china_map <- get_stadiamap(
  bbox = c(left = 95, bottom = 20, right = 125, top = 45),
  zoom = 4,
  maptype = "stamen_toner_lite"
)

# Reset the graphics device
try(dev.off(), silent = TRUE)

# Create the static 2D map with a heatmap, glowing points, and description
p5_full <- ggmap(china_map) +
  stat_density2d(
    data = china_cyber_data,
    aes(x = Longitude, y = Latitude, fill = after_stat(level), alpha = after_stat(level)),
    geom = "polygon",
    bins = 30,
    contour = TRUE
  ) +
  scale_fill_viridis(option = "inferno", direction = -1) +
  scale_alpha(range = c(0.2, 0.7)) +
  geom_point(
    data = china_cyber_data,
    aes(x = Longitude, y = Latitude, size = Successful_Attacks),
    color = "white",
    alpha = 0.9
  ) +
  geom_point(
    data = china_cyber_data,
    aes(x = Longitude, y = Latitude, size = Successful_Attacks),
    color = "yellow",
    alpha = 0.5
  ) +
  geom_text_repel(
    data = china_cyber_data,
    aes(x = Longitude, y = Latitude, label = paste0(City, " (", Successful_Attacks, ")")),
    color = "white",
    size = 4,
    box.padding = 0.5,
    segment.color = "white",
    segment.size = 0.5
  ) +
  labs(
    title = "Hotspots of Cyber Theft in China Targeting F-47 Technology",
    subtitle = "Map showing key cities in China with high cyber theft activity targeting F-47 NGAD technology (2025).",
    caption = "Intensity reflects the number of successful cyber attacks, with Beijing, Shanghai, and Shenzhen as primary sources of espionage risk.",
    x = "Longitude",
    y = "Latitude",
    fill = "Attack Density"
  ) +
  theme_minimal() +
  theme(
    plot.title = element_text(hjust = 0.5, face = "bold", color = "white"),
    plot.subtitle = element_text(hjust = 0.5, color = "white", size = 10),
    plot.caption = element_text(hjust = 0.5, color = "white", size = 8),
    plot.background = element_rect(fill = "black"),
    panel.background = element_rect(fill = "black"),
    legend.background = element_rect(fill = "black"),
    legend.text = element_text(color = "white"),
    legend.title = element_text(color = "white"),
    axis.text = element_text(color = "white"),
    axis.title = element_text(color = "white")
  )

# Attempt to print (failed with viewport error)
print(p5_full)

# Save the plot (worked to bypass display issue)
ggsave("china_cyber_hotspots.png", p5_full, width = 10, height = 8)
```

### Second Attempt: Using the `maps` Package
This approach used the `maps` package to create a static map, avoiding external API calls. It also encountered the same rendering issue but worked when saving directly to a file.

```R
# Load libraries
library(tidyverse)
library(viridis)
library(ggrepel)
library(maps)
library(mapdata)

# Clear the workspace and close all plot devices
rm(list = ls())
while (!is.null(dev.list())) dev.off()

# Dataset for cyber theft origins in China
china_cyber_data <- tibble(
  City = c("Beijing", "Shanghai", "Shenzhen", "Guangzhou", "Chengdu"),
  Successful_Attacks = c(120, 90, 80, 50, 40),
  Latitude = c(39.9042, 31.2304, 22.5431, 23.1291, 30.5728),
  Longitude = c(116.4074, 121.4737, 114.0579, 113.2644, 104.0668)
)

# Create the map
china_map_data <- map_data("world", region = "China")
p5_alt <- ggplot() +
  geom_polygon(data = china_map_data, aes(x = long, y = lat, group = group), fill = "grey80", color = "black") +
  geom_point(
    data = china_cyber_data,
    aes(x = Longitude, y = Latitude, size = Successful_Attacks, alpha = Successful_Attacks, color = Successful_Attacks),
    shape = 21,
    stroke = 0
  ) +
  scale_color_viridis(option = "inferno", direction = -1) +
  scale_alpha(range = c(0.3, 0.8)) +
  scale_size(range = c(5, 15)) +
  geom_point(
    data = china_cyber_data,
    aes(x = Longitude, y = Latitude, size = Successful_Attacks),
    color = "white",
    alpha = 0.9
  ) +
  geom_point(
    data = china_cyber_data,
    aes(x = Longitude, y = Latitude, size = Successful_Attacks),
    color = "yellow",
    alpha = 0.5
  ) +
  geom_text_repel(
    data = china_cyber_data,
    aes(x = Longitude, y = Latitude, label = paste0(City, " (", Successful_Attacks, ")")),
    color = "black",
    size = 4,
    box.padding = 0.5,
    segment.color = "black",
    segment.size = 0.5
  ) +
  labs(
    title = "Hotspots of Cyber Theft in China Targeting F-47 Technology",
    subtitle = "Map showing key cities in China with high cyber theft activity targeting F-47 NGAD technology (2025).",
    caption = "Intensity reflects the number of successful cyber attacks, with Beijing, Shanghai, and Shenzhen as primary sources of espionage risk.",
    x = "Longitude",
    y = "Latitude",
    color = "Successful Attacks",
    size = "Successful Attacks",
    alpha = "Successful Attacks"
  ) +
  theme_minimal() +
  theme(
    plot.title = element_text(hjust = 0.5, face = "bold"),
    plot.subtitle = element_text(hjust = 0.5, size = 10),
    plot.caption = element_text(hjust = 0.5, size = 8)
  ) +
  coord_fixed(ratio = 1.3)

# Save the plot directly without displaying (worked)
ggsave("china_cyber_hotspots_alt.png", p5_alt, width = 10, height = 8, dpi = 300)
```

### Final Successful Approach: Using `leaflet` with Stadia Maps
This approach used `leaflet` to create an interactive map, which successfully rendered and was saved as an HTML file, bypassing the RStudio display issues.

```R
# Load libraries
library(leaflet)
library(jsonlite)

# Clear the workspace and close all plot devices
rm(list = ls())
while (!is.null(dev.list())) dev.off()

# Dataset for cyber theft origins in China
china_cyber_data <- data.frame(
  City = c("Beijing", "Shanghai", "Shenzhen", "Guangzhou", "Chengdu"),
  Successful_Attacks = c(120, 90, 80, 50, 40),
  Latitude = c(39.9042, 31.2304, 22.5431, 23.1291, 30.5728),
  Longitude = c(116.4074, 121.4737, 114.0579, 113.2644, 104.0668),
  stringsAsFactors = FALSE
)

# Stadia Maps API key
api_key <- "9c644007-0572-4892-915a-8da356fe40ae"

# Create the Leaflet map
leaflet_map <- leaflet(data = china_cyber_data) %>%
  setView(lng = 110, lat = 35, zoom = 4) %>%
  addTiles(
    urlTemplate = paste0("https://tiles.stadiamaps.com/tiles/alidade_smooth/{z}/{x}/{y}{r}.png?api_key=", api_key),
    attribution = '© <a href="https://stadiamaps.com/">Stadia Maps</a>, © <a href="https://openmaptiles.org/">OpenMapTiles</a>, © <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a>',
    options = tileOptions(maxZoom = 20)
  ) %>%
  addCircleMarkers(
    lng = ~Longitude,
    lat = ~Latitude,
    radius = ~sqrt(Successful_Attacks) * 2,
    color = "yellow",
    fillColor = "red",
    fillOpacity = 0.7,
    stroke = TRUE,
    weight = 2,
    popup = ~paste0("<b>", City, "</b><br>Successful Attacks: ", Successful_Attacks)
  ) %>%
  addControl(
    html = paste0(
      "<h3 style='text-align: center;'>Hotspots of Cyber Theft in China Targeting F-47 Technology</h3>",
      "<p style='text-align: center; font-size: 14px;'>",
      "Map showing key cities in China with high cyber theft activity targeting F-47 NGAD technology (2025).<br>",
      "Intensity reflects the number of successful cyber attacks, with Beijing, Shanghai, and Shenzhen as primary sources of espionage risk.",
      "</p>"
    ),
    position = "topright"
  )

# Display the map
leaflet_map

# Save the map as an HTML file
htmlwidgets::saveWidget(leaflet_map, "china_cyber_hotspots.html", selfcontained = TRUE)
```

---

## Why It Matters

The F-47 NGAD fighter represents a pinnacle of military aviation technology, integrating advanced stealth, AI, and networked warfare capabilities. Protecting its intellectual property and operational data from cyber espionage is critical to maintaining a strategic advantage in global defense. This map highlights the significant threat posed by cyber theft operations in China, with cities like Beijing (120 successful attacks), Shanghai (90), and Shenzhen (80) identified as primary sources of risk in 2025. By visualizing these hotspots, defense stakeholders can prioritize cybersecurity measures, allocate resources effectively, and develop targeted strategies to safeguard the F-47 program. Understanding the geographical distribution of these threats is a crucial step in predictive analysis, ensuring the safety and integrity of next-generation military technology.

---

## Conclusion

This project successfully created an interactive map to visualize cyber theft hotspots in China targeting the F-47 NGAD technology, overcoming initial rendering challenges in RStudio by using the `leaflet` package. The map provides a clear, engaging way to present the data, with interactive features like popups and zooming that enhance stakeholder understanding. The inclusion of a detailed description ensures the audience grasps the context and significance of the data. While earlier attempts with `ggmap` and `maps` faced display issues, the final `leaflet` approach delivered a robust solution, saved as `china_cyber_hotspots.html` for easy sharing and presentation.

For future work, resolving the RStudio display issue (e.g., by reinstalling R/RStudio or checking the system graphics backend) would improve the workflow. Additionally, enhancing the map with features like a heatmap layer or custom icons could provide even deeper insights into the data.

---

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
```

---

### Steps to Set Up the Repository
1. **Create the Repository**:
   - Go to GitHub and create a new repository named `F47-Cyber-Espionage-Map`.
   - Initialize it with a README (you can overwrite it with the content above).

2. **Add Files**:
   - **README.md**: Copy the Markdown content above into the `README.md` file.
   - **china_cyber_hotspots.html**: Add the HTML file generated by the `leaflet` code to the repository. If you haven’t run the code yet, do so to generate the file, then upload it.
   - **LICENSE**: Add a simple MIT License file (optional but recommended for open-source projects):
     ```plaintext
     MIT License

     Copyright (c) 2025 [Your Name]

     Permission is hereby granted, free of charge, to any person obtaining a copy
     of this software and associated documentation files (the "Software"), to deal
     in the Software without restriction, including without limitation the rights
     to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
     copies of the Software, and to permit persons to whom the Software is
     furnished to do so, subject to the following conditions:

     The above copyright notice and this permission notice shall be included in all
     copies or substantial portions of the Software.

     THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
     IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
     FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
     AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
     LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
     OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
     SOFTWARE.
     ```

3. **Commit and Push**:
   - Commit the files to your repository and push them to GitHub.
   - Your repository will now be live, with the README providing a clear overview, the code, and the interactive map.

---

### Why This README is Compelling
- **Clear Structure**: The README is well-organized with sections for the overview, map instructions, code, "Why It Matters," and conclusion.
- **Comprehensive Code**: It includes all code attempts, showing the journey from initial failures to the final success, which is educational for others.
- **Interactive Map**: The `leaflet` map is highlighted as the key deliverable, with clear instructions for viewing it.
- **Why It Matters**: This section ties the visualization to the broader context of the F-47 NGAD program, emphasizing its importance in defense and cybersecurity.
- **Professional Tone**: The language is formal and suitable for a technical audience, such as defense analysts or stakeholders at Hitachi Digital Services.

---

### Next Steps
- **Share the Repository**: Share the GitHub repository link with your audience or stakeholders.
- **Embed the Map**: If you’re presenting this in a report or website, embed the `china_cyber_hotspots.html` file using an iframe, as mentioned earlier.
- **Resolve RStudio Issues**: For future projects, consider reinstalling R/RStudio or checking your system graphics backend to fix the display issue, as noted in the conclusion.
