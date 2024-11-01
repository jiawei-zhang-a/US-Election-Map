# US State Polling Map

This project visualizes the polling data for the 2024 US Presidential Election across all states, allowing users to interact with a dynamic map to view polling information, state leanings, and electoral college votes in real time. This project provides a data-driven, interactive map to help users understand the polling landscape and electoral college projections.

**Demo:** [US Election Map](https://us-election-map.netlify.app)


## Features

- **Interactive US Map**: Hover and click to view detailed polling data for each state.
- **Adjustable Toss-Up Margin**: Users can adjust the toss-up margin to redefine what constitutes a toss-up state. The map and electoral vote calculations will update dynamically based on this setting.
- **Pie Chart for State Polls**: Displays a pie chart for each state, showing the Democratic, Republican, and unknown vote distributions.
- **Electoral College Vote Bar**: A bar that visually represents the current electoral college standings based on polling data and toss-up adjustments.

## Data Sources and Calculation Method

All polling data displayed in this project is derived from multiple sources and personal recalculations:
  
- **2020 AP Vote Election Data**: Used as a baseline for some state leanings and historical comparison.
- **2020 Election Result Data**: Offers context and historical polling data trends.
- **Polling Data from [270toWin](https://www.270towin.com/2024-presidential-election-polls/)**: This data was used to inform polling estimates and trends for each state.
- **Self Prediction for Battleground States**: For key battleground states, projections were made based on a combination of recent polling data from various sources with shifts applied to account for anticipated changes in 2024.

> **Note**: All data presented in the project is recalculated to reflect a consistent, predictive model. The calculations and adjustments were made by the project author and are not an official projection.

## How to Use the Project

1. **Adjust the Toss-Up Margin**: Use the slider at the bottom to adjust the toss-up margin. This margin sets the polling percentage point difference threshold below which a state is considered a toss-up.
2. **Explore Individual States**: Click on any state to see detailed polling information, including the current lean and a pie chart of the polling distribution.
3. **View Electoral College Standing**: The electoral college bar updates based on polling data and user-selected toss-up margin, helping visualize the potential electoral landscape.

## Project Setup

To run the project locally, follow these steps:

### Prerequisites
- **Node.js** and **npm** installed on your machine.
- Familiarity with **Svelte** and **D3.js** libraries.

### Installation

1. Clone the repository:
   ```bash
   git clone https://github.com/jiawei-zhang-a/US-Election-Map.git
   cd US-Election-Map
   ```

2. Install dependencies:
    ```bash
    npm install
    ```

3. Start the development server:
    ```bash
    npm run dev
    ```

4. Open localhost in your browser to view the project.


## Code Overview

- **`src/App.svelte`**: Main component that renders the US map, electoral college bar, and toss-up margin slider.
- **`drawPieChart()`**: Renders the pie chart showing the distribution of polling data for selected states.
- **`getFillColor()`**: Determines the color of each state based on the polling margin and toss-up margin.
- **`renderElectoralCollegeBar()`**: Renders a dynamic bar that visualizes the electoral college standings based on the adjusted toss-up margin.

## Technologies Used

- **Svelte**: A front-end framework for building interactive user interfaces.
- **D3.js**: A JavaScript library for data-driven document manipulation, used for the map and pie chart visualizations.
- **TopoJSON**: To fetch and handle geographic data for rendering the US map.

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

---

By [Jiawei Zhang]

