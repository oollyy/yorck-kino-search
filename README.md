<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Yorck Cinema Planner</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        :root {
            --bg-color: #0d1117;
            --text-color: #e6edf3;
            --primary-color: #f7b731;
            --card-bg-color: rgba(31, 41, 55, 0.5);
            --border-color: rgba(55, 65, 81, 0.7);
            --input-bg-color: #161b22;
        }
        body {
            background-color: var(--bg-color);
            color: var(--text-color);
            background-image: radial-gradient(circle at 1px 1px, rgba(255,255,255,0.07) 1px, transparent 0);
            background-size: 2rem 2rem;
        }
        .glass-panel {
            background-color: var(--card-bg-color);
            backdrop-filter: blur(12px);
            -webkit-backdrop-filter: blur(12px);
            border: 1px solid var(--border-color);
            border-radius: 0.75rem;
            box-shadow: 0 4px 30px rgba(0, 0, 0, 0.1);
        }
        .form-element {
            background-color: var(--input-bg-color);
            border: 1px solid var(--border-color);
            border-radius: 0.5rem;
            padding: 0.75rem;
            transition: border-color 0.2s, box-shadow 0.2s;
        }
        .form-element:focus-within, .form-element:focus {
            outline: none;
            border-color: var(--primary-color);
            box-shadow: 0 0 0 2px rgba(247, 183, 49, 0.4);
        }
        .filter-toggle-group input:checked + label {
            background-color: var(--color-bg, var(--primary-color));
            color: var(--color-text, var(--bg-color));
            font-weight: 600;
            border-color: var(--color-bg, var(--primary-color));
        }
        .filter-toggle-group label {
            transition: all 0.2s ease-in-out;
            border: 1px solid var(--border-color);
        }
        .showtime-btn {
             transition: all 0.2s ease-in-out;
             border-width: 2px;
        }
        .showtime-btn:hover {
            background-color: var(--border-color);
        }
        .loader {
            border: 4px solid var(--border-color);
            border-top: 4px solid var(--primary-color);
            border-radius: 50%;
            width: 40px;
            height: 40px;
            animation: spin 1s linear infinite;
        }
        @keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }
        .lazy-load-placeholder {
            background-color: var(--border-color);
            border-radius: 0.375rem;
            width: 100%;
            height: 100%;
        }
        ::-webkit-scrollbar { width: 8px; }
        ::-webkit-scrollbar-track { background: var(--bg-color); }
        ::-webkit-scrollbar-thumb { background: var(--border-color); border-radius: 4px; }
        ::-webkit-scrollbar-thumb:hover { background: #4b5563; }
    </style>
</head>
<body class="font-sans">

    <div class="container mx-auto p-4 md:p-6 max-w-7xl">
        <header class="text-center mb-8">
            <h1 class="text-3xl md:text-4xl font-bold text-white mb-2">🎬 Yorck Cinema Planner</h1>
            <p class="text-gray-400">Your personal guide to the Yorck Kinos cinema program.</p>
        </header>

        <div class="glass-panel p-4 mb-6 flex flex-col md:flex-row items-center justify-between gap-4">
            <div class="flex flex-wrap items-center gap-4">
                <button id="fetch-btn" class="bg-amber-400 text-gray-900 font-bold py-2 px-5 rounded-lg hover:bg-amber-500 transition-colors duration-200">Update Listings</button>
                <button id="clear-cache-btn" class="bg-gray-700 text-white font-semibold py-2 px-5 rounded-lg hover:bg-red-600 transition-colors duration-200">Clear Cache</button>
                <div id="last-updated" class="text-sm text-gray-400"></div>
            </div>
            <div class="flex items-center gap-2">
                <label for="emoji-input" class="text-sm font-medium text-gray-300">Calendar Emoji:</label>
                <input type="text" id="emoji-input" value="🍿" class="form-element p-2 w-20 text-center">
                <button id="emoji-toggle-btn" title="Toggle 'Unsure' Emoji" class="bg-gray-700 text-white font-bold py-2 px-4 rounded-lg hover:bg-gray-600 transition-colors duration-200">❓</button>
            </div>
        </div>

        <div id="filters-panel" class="hidden glass-panel p-4 md:p-6 mb-6">
            <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6">
                <div>
                    <label for="search-input" class="block text-sm font-medium text-gray-300 mb-2">Search by Title</label>
                    <input type="text" id="search-input" placeholder="e.g., The Trial" class="form-element w-full">
                </div>
                <div>
                    <label for="cinema-filter" class="block text-sm font-medium text-gray-300 mb-2">Filter by Cinema</label>
                    <select id="cinema-filter" class="form-element w-full"></select>
                </div>
                <div>
                    <label for="date-filter" class="block text-sm font-medium text-gray-300 mb-2">Filter by Date</label>
                    <select id="date-filter" class="form-element w-full"></select>
                </div>
                <div>
                    <label for="group-by" class="block text-sm font-medium text-gray-300 mb-2">Group Results By</label>
                    <select id="group-by" class="form-element w-full">
                        <option value="date" selected>Date</option>
                        <option value="cinema">Cinema</option>
                        <option value="film">Film</option>
                    </select>
                </div>
            </div>
             <div class="grid grid-cols-1 md:grid-cols-2 gap-x-6 gap-y-4 mt-6">
                <div>
                    <label class="block text-sm font-medium text-gray-300 mb-2">Filter by Language Version</label>
                    <div id="version-filter" class="filter-toggle-group grid gap-2 grid-cols-2 sm:grid-cols-4"></div>
                </div>
                <div>
                    <label class="block text-sm font-medium text-gray-300 mb-2">Filter by Special Screenings</label>
                    <div id="specials-filter" class="filter-toggle-group grid gap-2 grid-cols-2 sm:grid-cols-4"></div>
                </div>
            </div>
            <div class="mt-6">
                <label class="block text-sm font-medium text-gray-300 mb-2">Filter by Time</label>
                <div id="time-filter" class="filter-toggle-group grid grid-cols-2 lg:grid-cols-4 gap-2"></div>
            </div>
        </div>

        <div id="status-container" class="text-center my-8"></div>
        <div id="results-container"></div>
    </div>

<script>
document.addEventListener('DOMContentLoaded', () => {
    const elements = {
        fetchBtn: document.getElementById('fetch-btn'),
        clearCacheBtn: document.getElementById('clear-cache-btn'),
        lastUpdatedEl: document.getElementById('last-updated'),
        emojiInput: document.getElementById('emoji-input'),
        emojiToggleBtn: document.getElementById('emoji-toggle-btn'),
        statusContainer: document.getElementById('status-container'),
        resultsContainer: document.getElementById('results-container'),
        filtersPanel: document.getElementById('filters-panel'),
        searchInput: document.getElementById('search-input'),
        cinemaFilter: document.getElementById('cinema-filter'),
        dateFilter: document.getElementById('date-filter'),
        versionFilterContainer: document.getElementById('version-filter'),
        specialsFilterContainer: document.getElementById('specials-filter'),
        timeFilterContainer: document.getElementById('time-filter'),
        groupByFilter: document.getElementById('group-by'),
    };

    const state = { allSessions: [], currentlyFilteredSessions: [], imageObserver: null };
    const CORS_PROXY = 'https://corsproxy.io/?';
    const YORCK_PROGRAM_URL = 'https://www.yorck.de/en/films';
    
    const VERSION_DEFINITIONS = {
        'DF': 'Deutsche Fassung (German Version)', 'OV': 'Originalfassung (Original Version)',
        'OmU': 'Original mit Untertiteln (Original with German Subtitles)',
        'OmeU': 'Original mit englischen Untertiteln (Original with English Subtitles)'
    };
    const STANDARD_VERSIONS = ['DF', 'OV', 'OmU', 'OmeU'];
    
    const COLOR_MAP = {
        'DF': { bg: '#3b82f6', text: '#ffffff' }, // Blue
        'OV': { bg: '#22c55e', text: '#ffffff' }, // Green
        'OmU': { bg: '#f97316', text: '#ffffff' }, // Orange
        'OmeU': { bg: '#a855f7', text: '#ffffff' }, // Purple
        'Mongay': { bg: '#ec4899', text: '#ffffff' }, // Pink
        'Creepy C': { bg: '#ef4444', text: '#ffffff' }, // Red
        'InTheMood': { bg: '#14b8a6', text: '#ffffff' }, // Teal
        'Sneak': { bg: '#0ea5e9', text: '#ffffff' }, // Sky Blue
        'Classic Sneak': { bg: '#6b7280', text: '#ffffff' }, // Gray
        'Fetch': { bg: '#d946ef', text: '#ffffff' }, // Fuchsia
        'Torment Nexus': { bg: '#65a30d', text: '#ffffff' }, // Lime
        'Ciné Club': { bg: '#0ea5e9', text: '#ffffff' }, // French Blue
        'Queerfilmnacht': { bg: '#c026d3', text: '#ffffff' }, // Magenta
        'FILM & TALK #2030': { bg: '#475569', text: '#ffffff' }, // Slate
        'Cine en Español': { bg: '#dc2626', text: '#ffffff' }, // Red
        'Crafts Club': { bg: '#f59e0b', text: '#ffffff' }, // Amber
        'Best of Cinema': { bg: '#8b5cf6', text: '#ffffff' }, // Violet
        'Bis(s) zum Abspann': { bg: '#b91c1c', text: '#ffffff' }, // Dark Red
        'Boulevard Noir': { bg: '#4f46e5', text: '#ffffff' }, // Indigo
        'Screen Legends': { bg: '#374151', text: '#ffffff' }, // Cool Gray
        'SV': { bg: '#78716c', text: '#ffffff' }, // Stone
        'Festival': { bg: '#d97706', text: '#ffffff' }, // Amber
        'KF': { bg: '#06b6d4', text: '#ffffff' }, // Cyan
        'Lesung': { bg: '#a16207', text: '#ffffff'}, // Yellow-Brown
        'default': { border: '#4b5563', text: '#d1d5db' }
    };


    function initializeApp() {
        setupEventListeners();
        setupImageObserver();
        const storedData = localStorage.getItem('yorckData');
        if (storedData) {
            try {
                const { films, timestamp } = JSON.parse(storedData);
                processRawData(films);
                elements.lastUpdatedEl.textContent = `Last Updated: ${new Date(timestamp).toLocaleString()}`;
                setupFilters();
                elements.filtersPanel.classList.remove('hidden');
                renderFilteredResults(); // Automatically render on load
            } catch (e) {
                localStorage.removeItem('yorckData');
                showStatus('Could not load stored data. Please update listings.', 'error');
            }
        } else {
            showStatus('No data found. Click "Update Listings" to fetch the cinema program.', 'info');
        }
    }
    
    function setupEventListeners() {
        elements.fetchBtn.addEventListener('click', handleFetchAll);
        elements.clearCacheBtn.addEventListener('click', () => { localStorage.removeItem('yorckData'); location.reload(); });
        elements.emojiToggleBtn.addEventListener('click', () => {
            const currentVal = elements.emojiInput.value;
            elements.emojiInput.value = currentVal.startsWith('❓') ? currentVal.substring(1) : '❓' + currentVal;
        });

        const debouncedRender = debounce(renderFilteredResults, 250);
        elements.searchInput.addEventListener('input', debouncedRender);
        [elements.cinemaFilter, elements.dateFilter, elements.groupByFilter, elements.versionFilterContainer, elements.specialsFilterContainer, elements.timeFilterContainer]
            .forEach(el => el.addEventListener('change', renderFilteredResults));
        elements.resultsContainer.addEventListener('click', handleShowtimeClick);
    }

    function setupImageObserver() {
        state.imageObserver = new IntersectionObserver((entries, observer) => {
            entries.forEach(entry => {
                if (entry.isIntersecting) {
                    const placeholder = entry.target;
                    const img = document.createElement('img');
                    img.src = placeholder.dataset.src;
                    img.alt = placeholder.dataset.alt;
                    img.className = 'rounded-md w-full h-full object-cover';
                    img.onload = () => placeholder.parentNode.replaceChild(img, placeholder);
                    img.onerror = () => { placeholder.style.backgroundColor = '#374151'; }
                    observer.unobserve(placeholder);
                }
            });
        }, { rootMargin: "300px 0px" });
    }

    async function handleFetchAll() {
        showStatus('', 'loading');
        elements.fetchBtn.disabled = true;
        elements.fetchBtn.textContent = 'Updating...';
        try {
            const response = await fetch(`${CORS_PROXY}${encodeURIComponent(YORCK_PROGRAM_URL)}`);
            if (!response.ok) throw new Error(`Network error: ${response.status}`);
            
            const htmlText = await response.text();
            const doc = new DOMParser().parseFromString(htmlText, 'text/html');
            const nextDataScript = doc.getElementById('__NEXT_DATA__');
            if (!nextDataScript) throw new Error('Scraping failed: Could not find data script.');
            
            const films = JSON.parse(nextDataScript.textContent).props.pageProps.films;
            if (!films) throw new Error('Scraping failed: Film data missing.');
            
            localStorage.setItem('yorckData', JSON.stringify({ films, timestamp: new Date().toISOString() }));
            
            processRawData(films);
            
            elements.lastUpdatedEl.textContent = `Updated: ${new Date().toLocaleString()}`;
            showStatus('Listings updated successfully!', 'success');
            setTimeout(() => elements.statusContainer.innerHTML = '', 3000);
            
            setupFilters();
            renderFilteredResults();
            elements.filtersPanel.classList.remove('hidden');
        } catch (error) {
            console.error('Error fetching data:', error);
            showStatus(`Failed to update listings. Error: ${error.message}`, 'error');
        } finally {
            elements.fetchBtn.disabled = false;
            elements.fetchBtn.textContent = 'Update Listings';
        }
    }

    function processRawData(films) {
        state.allSessions = films.flatMap(film => 
            (film.fields.sessions || []).map(sessionRef => ({
                film: film.fields,
                session: sessionRef.fields,
                cinema: sessionRef.fields.cinema?.fields
            })).filter(item => item.cinema)
        );
    }

    function setupFilters() {
        const createToggleGroup = (items, name, tooltips = {}) => items.map(item => {
            const colorStyle = COLOR_MAP[item] ? `--color-bg: ${COLOR_MAP[item].bg}; --color-text: ${COLOR_MAP[item].text};` : '';
            return `
            <div>
                <input type="checkbox" id="${name}-${item}" value="${item}" class="hidden">
                <label for="${name}-${item}" class="block cursor-pointer rounded-md py-2 px-4 text-center" style="${colorStyle}" title="${tooltips[item] || ''}">${item}</label>
            </div>`;
        }).join('');

        const createRadioGroup = (items, name) => items.map((item, index) => `
            <div>
                <input type="radio" id="${name}-${item.value}" name="${name}" value="${item.value}" class="hidden" ${index === 0 ? 'checked' : ''}>
                <label for="${name}-${item.value}" class="block cursor-pointer rounded-md py-2 px-4 text-center">${item.label}</label>
            </div>`).join('');

        elements.timeFilterContainer.innerHTML = createRadioGroup([
            { value: 'all', label: 'All Day' }, { value: 'morning', label: 'Morning (til 12:00)' },
            { value: 'afternoon', label: 'Afternoon (12-17:00)' }, { value: 'evening', label: 'Evening (from 17:00)' }
        ], 'time');

        const uniqueCinemas = [...new Set(state.allSessions.map(s => s.cinema.name.trim()))].sort();
        elements.cinemaFilter.innerHTML = '<option value="all">All Cinemas</option>' + uniqueCinemas.map(c => `<option value="${c}">${c}</option>`).join('');

        const uniqueDates = [...new Set(state.allSessions.map(s => s.session.startTime.split('T')[0]))].sort();
        elements.dateFilter.innerHTML = '<option value="all">All Dates</option>' + uniqueDates.map(dateStr => {
            const dateObj = new Date(dateStr + 'T00:00:00Z');
            return `<option value="${dateStr}">${dateObj.toLocaleDateString('en-US', { weekday: 'short', month: 'short', day: 'numeric', timeZone: 'UTC' })}</option>`;
        }).join('');

        const allFormats = [...new Set(state.allSessions.flatMap(s => s.session.formats || []))];
        const specialFormats = allFormats.filter(f => !STANDARD_VERSIONS.includes(f)).sort();
        elements.versionFilterContainer.innerHTML = createToggleGroup(STANDARD_VERSIONS.filter(v => allFormats.includes(v)), 'version', VERSION_DEFINITIONS);
        elements.specialsFilterContainer.innerHTML = createToggleGroup(specialFormats, 'special');
    }
    
    function renderFilteredResults() {
        const filters = {
            search: elements.searchInput.value.toLowerCase(),
            cinema: elements.cinemaFilter.value,
            date: elements.dateFilter.value,
            versions: [...elements.versionFilterContainer.querySelectorAll(':checked')].map(cb => cb.value),
            specials: [...elements.specialsFilterContainer.querySelectorAll(':checked')].map(cb => cb.value),
            time: elements.timeFilterContainer.querySelector(':checked')?.value || 'all',
        };

        state.currentlyFilteredSessions = state.allSessions.filter(item => {
            const sessionTime = new Date(item.session.startTime);
            const sessionHour = sessionTime.getHours();
            const itemFormats = item.session.formats || [];
            
            const allVersionFilters = [...filters.versions, ...filters.specials];

            return (
                (!filters.search || item.film.title.toLowerCase().includes(filters.search)) &&
                (filters.cinema === 'all' || item.cinema.name.trim() === filters.cinema) &&
                (filters.date === 'all' || item.session.startTime.startsWith(filters.date)) &&
                (allVersionFilters.length === 0 || itemFormats.some(v => allVersionFilters.includes(v))) &&
                !((filters.time === 'morning' && sessionHour >= 12) ||
                  (filters.time === 'afternoon' && (sessionHour < 12 || sessionHour >= 17)) ||
                  (filters.time === 'evening' && sessionHour < 17))
            );
        });
        
        elements.statusContainer.innerHTML = '';
        elements.resultsContainer.innerHTML = '';

        if (state.currentlyFilteredSessions.length === 0) {
            return showStatus('No showtimes match your criteria.', 'info');
        }
        
        const groupBy = elements.groupByFilter.value;
        const renderFunction = { date: renderGroupedByDate, cinema: renderGroupedByCinema, film: renderGroupedByFilm }[groupBy];
        renderFunction(state.currentlyFilteredSessions);
        
        elements.resultsContainer.querySelectorAll('.lazy-load-placeholder').forEach(el => state.imageObserver.observe(el));
    }
    
    function createShowtimeButtons(sessions, indexMap) {
        return sessions
            .sort((a, b) => new Date(a.session.startTime) - new Date(b.session.startTime))
            .map(item => {
                const time = new Date(item.session.startTime);
                const format = (item.session.formats || [])[0]; // Use first format for color
                const color = COLOR_MAP[format]?.bg || COLOR_MAP['default'].border;

                return `<button class="showtime-btn border-gray-700 bg-gray-900/50 rounded-lg p-2 text-center w-24" style="border-color:${color}" data-session-index="${indexMap.get(item)}">
                    <span class="font-bold text-lg">${time.toLocaleTimeString('en-GB', { hour: '2-digit', minute: '2-digit' })}</span>
                    <span class="block text-xs text-gray-400">${(item.session.formats || []).join(', ')}</span>
                </button>`;
            }).join('');
    }

    const createFilmEntryHTML = (film, sessions, indexMap) => {
        const posterUrl = film.heroImage?.fields?.image?.fields?.file?.url ? `https:${film.heroImage.fields.image.fields.file.url}` : 'https://via.placeholder.com/320x180.png?text=No+Image';
        const filmUrl = `https://www.yorck.de/en/films/${film.slug}`;
        const details = [film.mainLabel, film.runtime ? `${film.runtime} min` : null, film.fsk !== null ? `FSK ${film.fsk}` : null].filter(Boolean).join(' | ');

        return `
            <div class="flex flex-col sm:flex-row gap-4 py-4 border-b border-gray-700 last:border-b-0">
                <a href="${filmUrl}" target="_blank" rel="noopener noreferrer" class="flex-shrink-0 w-full sm:w-64 aspect-video">
                    <div class="lazy-load-placeholder" data-src="${posterUrl}" data-alt="${film.title} Poster"></div>
                </a>
                <div class="flex-grow">
                    <a href="${filmUrl}" target="_blank" rel="noopener noreferrer" class="hover:underline">
                        <h5 class="text-lg font-bold text-white">${film.title}</h5>
                    </a>
                    <p class="text-sm text-gray-400 mt-1 italic">${film.tagline || ''}</p>
                    <p class="text-xs text-gray-300 font-medium mt-2">${details}</p>
                    <div class="flex flex-wrap gap-2 mt-3">${createShowtimeButtons(sessions, indexMap)}</div>
                </div>
            </div>`;
    };

    const renderGroupedBy = (sessions, primaryKeyFn, secondaryKeyFn, primaryHeaderFn, secondaryHeaderFn) => {
        const indexMap = new Map(sessions.map((s, i) => [s, i]));
        const groupedPrimary = sessions.reduce((acc, item) => { (acc[primaryKeyFn(item)] = acc[primaryKeyFn(item)] || []).push(item); return acc; }, {});
        const html = Object.keys(groupedPrimary).sort().map(primaryKey => {
            const secondaryHTML = Object.entries(groupedPrimary[primaryKey].reduce((acc, item) => {
                (acc[secondaryKeyFn(item)] = acc[secondaryKeyFn(item)] || []).push(item); return acc;
            }, {})).sort(([a], [b]) => a.localeCompare(b)).map(([secondaryKey, secondaryItems]) => {
                const filmsHTML = Object.values(secondaryItems.reduce((acc, item) => {
                    if (!acc[item.film.title]) acc[item.film.title] = { film: item.film, sessions: [] };
                    acc[item.film.title].sessions.push(item);
                    return acc;
                }, {})).sort((a,b) => a.film.title.localeCompare(b.film.title))
                .map(({film, sessions}) => createFilmEntryHTML(film, sessions, indexMap)).join('');
                return `<div class="glass-panel p-4 mb-4">${secondaryHeaderFn(secondaryKey)}${filmsHTML}</div>`;
            }).join('');
            return primaryHeaderFn(primaryKey) + secondaryHTML;
        }).join('');
        elements.resultsContainer.innerHTML = html;
    };
    
    const dateHeader = date => `<h3 class="text-2xl font-semibold text-white border-b-2 border-amber-400 pb-2 mb-6 mt-8">${new Date(date + 'T00:00:00Z').toLocaleDateString('en-US', { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric', timeZone: 'UTC' })}</h3>`;
    const cinemaHeader = name => `<h3 class="text-2xl font-semibold text-white border-b-2 border-amber-400 pb-2 mb-6 mt-8">${name}</h3>`;
    const cinemaSubHeader = name => `<h4 class="text-xl font-bold text-amber-400 mb-4">${name}</h4>`;
    const dateSubHeader = date => `<h4 class="text-xl font-bold text-amber-400 mb-4">${new Date(date + 'T00:00:00Z').toLocaleDateString('en-US', { weekday: 'long', month: 'long', day: 'numeric', timeZone: 'UTC' })}</h4>`;
    
    const renderGroupedByDate = s => renderGroupedBy(s, i => i.session.startTime.split('T')[0], i => i.cinema.name.trim(), dateHeader, cinemaSubHeader);
    const renderGroupedByCinema = s => renderGroupedBy(s, i => i.cinema.name.trim(), i => i.session.startTime.split('T')[0], cinemaHeader, dateSubHeader);

    function renderGroupedByFilm(sessions) {
        const indexMap = new Map(sessions.map((s, i) => [s, i]));
        const groupedByFilm = sessions.reduce((acc, item) => { (acc[item.film.title] = acc[item.film.title] || []).push(item); return acc; }, {});
        const html = Object.keys(groupedByFilm).sort().map(filmTitle => {
            const filmSessions = groupedByFilm[filmTitle];
            const filmData = filmSessions[0].film;
            const posterUrl = filmData.heroImage?.fields?.image?.fields?.file?.url ? `https:${filmData.heroImage.fields.image.fields.file.url}` : 'https://via.placeholder.com/320x180.png?text=No+Image';
            const details = [filmData.mainLabel, filmData.runtime ? `${filmData.runtime} min` : null, filmData.fsk !== null ? `FSK ${filmData.fsk}` : null].filter(Boolean).join(' | ');

            let filmHTML = `
                <div class="flex flex-col sm:flex-row gap-4 py-4 border-b-2 border-amber-400 mt-8">
                    <a href="https://www.yorck.de/en/films/${filmData.slug}" target="_blank" rel="noopener noreferrer" class="flex-shrink-0 w-full sm:w-64 aspect-video">
                        <div class="lazy-load-placeholder" data-src="${posterUrl}" data-alt="${filmTitle} Poster"></div>
                    </a>
                    <div class="flex-grow">
                        <a href="https://www.yorck.de/en/films/${filmData.slug}" target="_blank" rel="noopener noreferrer" class="hover:underline">
                            <h3 class="text-2xl font-semibold text-white">${filmTitle}</h3>
                        </a>
                        <p class="text-sm text-gray-400 mt-1 italic">${filmData.tagline || ''}</p>
                        <p class="text-xs text-gray-300 font-medium mt-2">${details}</p>
                    </div>
                </div>`;
            
            const groupedByDate = filmSessions.reduce((acc, item) => {
                const date = item.session.startTime.split('T')[0];
                if (!acc[date]) acc[date] = []; acc[date].push(item);
                return acc;
            }, {});

            Object.keys(groupedByDate).sort().forEach(date => {
                filmHTML += `<h4 class="text-lg font-bold text-amber-400 mt-4 mb-2">${new Date(date + 'T00:00:00Z').toLocaleDateString('en-US', { weekday: 'long', month: 'long', day: 'numeric', timeZone: 'UTC' })}</h4>`;
                const groupedByCinema = groupedByDate[date].reduce((acc, item) => {
                    if (!acc[item.cinema.name]) acc[item.cinema.name] = []; acc[item.cinema.name].push(item);
                    return acc;
                }, {});

                Object.keys(groupedByCinema).sort().forEach(cinemaName => {
                    filmHTML += `<div class="pl-4 border-l-2 border-gray-700 mb-3"><p class="font-semibold">${cinemaName}</p><div class="flex flex-wrap gap-2 mt-1">${createShowtimeButtons(groupedByCinema[cinemaName], indexMap)}</div></div>`;
                });
            });
            return filmHTML;
        }).join('');
        elements.resultsContainer.innerHTML = html;
    }

    function handleShowtimeClick(e) {
        const btn = e.target.closest('.showtime-btn');
        if (!btn || !btn.dataset.sessionIndex) return;
        const item = state.currentlyFilteredSessions[parseInt(btn.dataset.sessionIndex, 10)];
        if (!item) { console.error('Session data not found'); return; }
        downloadIcsFile(generateIcsContent(item.film, item.session, item.cinema.name), item.film.slug, item.session.startTime);
    }

    function generateIcsContent(film, session, cinemaName) {
        const startDate = new Date(session.startTime);
        const endDate = new Date(startDate.getTime() + (film.runtime || 90) * 60000);
        const now = new Date();
        const emoji = elements.emojiInput.value || '🍿';

        const formatIcsDate = (date) => date.toISOString().replace(/[-:]/g, '').split('.')[0] + 'Z';
        const formatIcsDateWithTZ = (date) => {
            const pad = (n) => String(n).padStart(2, '0');
            return `${date.getFullYear()}${pad(date.getMonth() + 1)}${pad(date.getDate())}T${pad(date.getHours())}${pad(date.getMinutes())}${pad(date.getSeconds())}`;
        };
        const escapeIcsText = (text) => (text || '').replace(/\\/g, '\\\\').replace(/;/g, '\\;').replace(/,/g, '\\,').replace(/\n/g, '\\n');

        const description = `Synopsis: ${escapeIcsText(film.tagline)}\\n\\nRuntime: ${film.runtime || 'N/A'} min\\nVersion: ${(session.formats || []).join(', ')}`;
        const filmUrl = `https://www.yorck.de/en/films/${film.slug}`;
        
        const vTimezone = [
            'BEGIN:VTIMEZONE', 'TZID:Europe/Berlin', 'BEGIN:DAYLIGHT', 'TZOFFSETFROM:+0100', 'TZOFFSETTO:+0200', 'TZNAME:CEST', 'DTSTART:19700329T020000', 'RRULE:FREQ=YEARLY;BYDAY=-1SU;BYMONTH=3', 'END:DAYLIGHT',
            'BEGIN:STANDARD', 'TZOFFSETFROM:+0200', 'TZOFFSETTO:+0100', 'TZNAME:CET', 'DTSTART:19701025T030000', 'RRULE:FREQ=YEARLY;BYDAY=-1SU;BYMONTH=10', 'END:STANDARD', 'END:VTIMEZONE'
        ].join('\r\n');

        return [
            'BEGIN:VCALENDAR', 'VERSION:2.0', 'PRODID:-//YorckPlanner//NONSGML v2.3//EN', vTimezone,
            'BEGIN:VEVENT', `UID:${now.getTime()}-${film.vistaId}@yorck.de`, `DTSTAMP:${formatIcsDate(now)}`,
            `DTSTART;TZID=Europe/Berlin:${formatIcsDateWithTZ(startDate)}`,
            `DTEND;TZID=Europe/Berlin:${formatIcsDateWithTZ(endDate)}`,
            `SUMMARY:${escapeIcsText(`${emoji} ${film.title}`)}`, `DESCRIPTION:${description}`,
            `LOCATION:${escapeIcsText(cinemaName)}`, `URL;VALUE=URI:${filmUrl}`, 'END:VEVENT', 'END:VCALENDAR'
        ].join('\r\n');
    }

    function downloadIcsFile(content, filmSlug, startTime) {
        const date = new Date(startTime);
        const [y, m, d, h, min] = [date.getFullYear(), String(date.getMonth() + 1).padStart(2, '0'), String(date.getDate()).padStart(2, '0'), String(date.getHours()).padStart(2, '0'), String(date.getMinutes()).padStart(2, '0')];
        const safeSlug = (filmSlug || 'film').replace(/[^a-z0-9]/gi, '-').toLowerCase();
        const filename = `${y}-${m}-${d}-${h}${min}-${safeSlug}.ics`;

        const blob = new Blob([content], { type: 'text/calendar;charset=utf-8' });
        const link = document.createElement('a');
        link.href = URL.createObjectURL(blob);
        link.download = filename;
        document.body.appendChild(link);
        link.click();
        document.body.removeChild(link);
        URL.revokeObjectURL(link.href);
    }
    
    function showStatus(message, type = 'info') {
        elements.statusContainer.innerHTML = '';
        if (!message && type !== 'loading') return;
        const colors = { error: 'text-red-400 bg-red-900/50', success: 'text-green-300 bg-green-900/50', info: 'text-blue-300 bg-blue-900/50' };
        elements.statusContainer.innerHTML = type === 'loading' ? `<div class="loader mx-auto"></div>` : `<p class="${colors[type]} p-3 rounded-lg">${message}</p>`;
    }
    
    function debounce(func, delay) {
        let timeout;
        return function(...args) { clearTimeout(timeout); timeout = setTimeout(() => func.apply(this, args), delay); };
    }

    initializeApp();
});
</script>

</body>
</html>
