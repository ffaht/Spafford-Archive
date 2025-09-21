---
layout: default
title: Jam Chart
permalink: /jam-chart/
---

<div id="jam-chart">
    <div class="search-container">
        <input type="text" id="search-input" placeholder="Search songs, venues, dates..." />
        <select id="filter-select">
            <option value="all">All Jams</option>
            <option value="highlighted">Highlighted Only</option>
        </select>
    </div>
    
    <div class="jam-table-container">
        <table class="jam-table">
            <thead>
                <tr>
                    <th class="date-col sortable" data-column="date">
                        Date <span class="sort-indicator">▼</span>
                    </th>
                    <th class="song-col sortable" data-column="song">
                        Song <span class="sort-indicator"></span>
                    </th>
                    <th class="timing-col sortable" data-column="timing">
                        Timing <span class="sort-indicator"></span>
                    </th>
                    <th class="venue-col sortable" data-column="venue">
                        Venue <span class="sort-indicator"></span>
                    </th>
                    <th class="notes-col sortable" data-column="notes">
                        Notes <span class="sort-indicator"></span>
                    </th>
                </tr>
            </thead>
            <tbody id="jam-table-body">
                </tbody>
        </table>
    </div>
</div>

<script>
let jamData = [];
let filteredJams = [];
let currentSort = { column: 'date', direction: 'desc' };

// Load jam data
fetch('{{ "/assets/data/jams.json" | relative_url }}')
    .then(response => response.json())
    .then(data => {
        jamData = data;
        filteredJams = [...jamData];
        sortData('date', 'desc');
        renderJamTable();
        setupEventListeners();
        handleUrlParams();
    })
    .catch(error => console.error('Error loading jam data:', error));

function sortData(column, direction = null) {
    if (direction === null) {
        if (currentSort.column === column) {
            direction = currentSort.direction === 'asc' ? 'desc' : 'asc';
        } else {
            direction = 'asc';
        }
    }
    
    currentSort = { column, direction };
    
    filteredJams.sort((a, b) => {
        let valueA = a[column] || '';
        let valueB = b[column] || '';
        
        if (column === 'date') {
            valueA = new Date(valueA + 'T00:00:00');
            valueB = new Date(valueB + 'T00:00:00');
        } else if (column === 'timing') {
            // Convert timing to seconds for proper sorting
            valueA = timingToSeconds(valueA);
            valueB = timingToSeconds(valueB);
        } else if (column === 'location') {
            valueA = ((a.city || '') + ' ' + (a.state || '')).trim().toLowerCase();
            valueB = ((b.city || '') + ' ' + (b.state || '')).trim().toLowerCase();
        } else {
            valueA = valueA.toString().toLowerCase();
            valueB = valueB.toString().toLowerCase();
        }
        
        let comparison = 0;
        if (valueA < valueB) comparison = -1;
        else if (valueA > valueB) comparison = 1;
        
        return direction === 'asc' ? comparison : -comparison;
    });
    
    updateSortIndicators();
}

function timingToSeconds(timing) {
    if (!timing || timing === '') return 0;
    
    // If it contains a colon, parse as mm:ss
    if (timing.includes(':')) {
        const parts = timing.split(':');
        if (parts.length === 2) {
            const minutes = parseInt(parts[0]) || 0;
            const seconds = parseInt(parts[1]) || 0;
            return minutes * 60 + seconds;
        }
    }
    
    // Otherwise try to parse as plain number (seconds)
    return parseInt(timing) || 0;
}

function updateSortIndicators() {
    document.querySelectorAll('.sort-indicator').forEach(indicator => {
        indicator.textContent = '';
    });
    const currentHeader = document.querySelector(`[data-column="${currentSort.column}"] .sort-indicator`);
    if (currentHeader) {
        currentHeader.textContent = currentSort.direction === 'asc' ? '▲' : '▼';
    }
}

function setupEventListeners() {
    // Column sorting
    document.querySelectorAll('.sortable').forEach(header => {
        header.addEventListener('click', () => {
            const column = header.dataset.column;
            sortData(column);
            renderJamTable();
        });
    });
    
    // Search and filter
    document.getElementById('search-input').addEventListener('input', filterJams);
    document.getElementById('filter-select').addEventListener('change', filterJams);
    
    // Clickable filters (using event delegation)
    document.getElementById('jam-table-body').addEventListener('click', (e) => {
        if (e.target.classList.contains('clickable-filter')) {
            const value = e.target.dataset.value;
            document.getElementById('search-input').value = value;
            filterJams();
        }
    });
}

function renderJamTable() {
    const tableBody = document.getElementById('jam-table-body');
    tableBody.innerHTML = '';
    
    if (filteredJams.length === 0) {
        tableBody.innerHTML = '<tr><td colspan="6" class="no-results">No jams found.</td></tr>';
        return;
    }
    
    filteredJams.forEach(jam => {
        const row = document.createElement('tr');
        if (jam.highlighted) {
            row.classList.add('highlighted-row');
        }
        
        // Format date for display
        let displayDate = jam.date;
        try {
            const dateObj = new Date(jam.date + 'T00:00:00');
            displayDate = dateObj.toLocaleDateString('en-US', { 
                year: 'numeric', 
                month: '2-digit', 
                day: '2-digit' 
            });
        } catch (e) {
            // Keep original date if parsing fails
        }
        
        // Combine city and state for location
        let location = '';
        if (jam.city) {
            location = jam.city;
            if (jam.state) {
                location += ', ' + jam.state;
            }
        }
        
        row.innerHTML = `
            <td class="date-cell">
                <span class="clickable-filter" data-value="${jam.date}">${displayDate}</span>
            </td>
            <td class="song-cell ${jam.highlighted ? 'highlighted-jam' : ''}">
                <span class="clickable-filter" data-value="${jam.song}">${jam.song}</span>
            </td>
            <td class="timing-cell">${jam.timing || ''}</td>
            <td class="venue-cell">
                <span class="clickable-filter" data-value="${jam.venue}">${jam.venue}</span>
            </td>
            <td class="notes-cell">${jam.notes || ''}</td>
        `;
        
        tableBody.appendChild(row);
    });
}

function filterJams() {
    const searchTerm = document.getElementById('search-input').value.toLowerCase();
    const filter = document.getElementById('filter-select').value;
    
    filteredJams = jamData.filter(jam => {
        const matchesSearch = searchTerm === '' || 
                            jam.song.toLowerCase().includes(searchTerm) ||
                            jam.venue.toLowerCase().includes(searchTerm) ||
                            jam.date.includes(searchTerm) ||
                            (jam.city && jam.city.toLowerCase().includes(searchTerm)) ||
                            (jam.state && jam.state.toLowerCase().includes(searchTerm)) ||
                            (jam.timing && jam.timing.toLowerCase().includes(searchTerm));
        
        const matchesFilter = filter === 'all' || (filter === 'highlighted' && jam.highlighted);
        
        return matchesSearch && matchesFilter;
    });
    
    // Re-sort after filtering
    sortData(currentSort.column, currentSort.sortDirection);
    renderJamTable();
}

function handleUrlParams() {
    const urlParams = new URLSearchParams(window.location.search);
    const filterParam = urlParams.get('filter');
    if (filterParam) {
        document.getElementById('search-input').value = filterParam;
        setTimeout(() => { 
            filterJams(); 
        }, 100);
    }
}
</script>

<style>
.search-container {
    display: flex;
    gap: 1rem;
    margin-bottom: 2rem;
    align-items: center;
}

#search-input {
    flex: 1;
    padding: 0.75rem;
    border: 1px solid var(--border-color);
    border-radius: 8px;
    background: var(--card-bg);
    color: var(--text-color);
    font-size: 1rem;
}

#filter-select {
    padding: 0.75rem;
    border: 1px solid var(--border-color);
    border-radius: 8px;
    background: var(--card-bg);
    color: var(--text-color);
}

.jam-table-container {
    background: var(--card-bg);
    border-radius: 8px;
    border: 1px solid var(--border-color);
    overflow-x: auto;
}

.jam-table {
    width: 100%;
    border-collapse: collapse;
    font-size: 0.9rem;
}

.jam-table th {
    background: #0a0a0a;
    color: var(--text-color);
    font-weight: 600;
    padding: 1rem 0.75rem;
    text-align: left;
    border-bottom: 2px solid var(--border-color);
    position: sticky;
    top: 0;
    z-index: 10;
}

.sortable {
    cursor: pointer;
    user-select: none;
    transition: background-color 0.2s;
}

.sortable:hover {
    background: #151515;
}

.sort-indicator {
    display: inline-block;
    margin-left: 0.5rem;
    font-size: 0.8rem;
    color: var(--yellow);
}

.jam-table td {
    padding: 0.75rem;
    border-bottom: 1px solid var(--border-color);
    vertical-align: top;
    line-height: 1.4;
}

.jam-table tr:hover {
    background: rgba(255, 255, 255, 0.02);
}

.highlighted-row {
    border-left: 4px solid var(--yellow);
}

.highlighted-row td:first-child {
    border-left: none;
}

/* Column sizing */
.date-col { width: 10%; min-width: 90px; }
.song-col { width: 15%; min-width: 120px; }
.timing-col { width: 10%; min-width: 80px; }
.venue-col { width: 25%; min-width: 150px; }
.location-col { width: 15%; min-width: 120px; }
.notes-col { width: 25%; min-width: 200px; }

.date-cell {
    font-family: monospace;
    font-size: 0.85rem;
    color: var(--muted-color);
}

.song-cell {
    font-weight: 500;
    color: var(--text-color);
}

.timing-cell {
    font-family: monospace;
    font-size: 0.85rem;
    color: var(--muted-color);
}

.venue-cell {
    color: var(--text-color);
}

.location-cell {
    color: var(--muted-color);
    font-size: 0.85rem;
}

.notes-cell {
    color: var(--text-color);
    font-size: 0.85rem;
    line-height: 1.3;
}

.highlighted-jam {
    color: var(--yellow) !important;
    font-weight: bold !important;
}

.clickable-filter {
    cursor: pointer;
    text-decoration: underline;
    text-decoration-color: transparent;
    transition: all 0.2s ease;
}

.clickable-filter:hover {
    text-decoration-color: var(--yellow);
    color: var(--yellow);
}

.no-results {
    text-align: center;
    color: var(--muted-color);
    font-style: italic;
    padding: 2rem;
}

@media (max-width: 1024px) {
    .jam-table {
        font-size: 0.8rem;
    }
    
    .jam-table th,
    .jam-table td {
        padding: 0.5rem 0.4rem;
    }
}

@media (max-width: 768px) {
    .search-container {
        flex-direction: column;
    }
    
    .jam-table-container {
        border-radius: 4px;
    }
    
    .jam-table th,
    .jam-table td {
        padding: 0.4rem 0.3rem;
    }
    
    .notes-cell {
        font-size: 0.75rem;
    }
    
    /* Hide timing column on mobile */
    .timing-col,
    .timing-cell {
        display: none;
    }
}
</style>