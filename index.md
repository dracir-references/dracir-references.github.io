---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: page
---

# References

{% assign recipes = site.references | sort: "title" %}
<section class="recipe-catalog" data-recipe-catalog>
	<div class="d-flex flex-wrap align-items-center justify-content-between gap-3 mb-3">
		<div class="d-flex align-items-center gap-2">
			<button
				type="button"
				class="btn btn-outline-secondary btn-sm"
				data-bs-toggle="collapse"
				data-bs-target="#recipe-filter-panel"
				aria-expanded="false"
				aria-controls="recipe-filter-panel"
			>
				Show filters
			</button>
			<p class="small text-muted mb-0" data-results-count>{{ recipes.size }} recipes</p>
		</div>
	</div>

	<div class="collapse mb-4" id="recipe-filter-panel" data-filter-panel>
		<div class="card p-4 catalog-panel">
			<div class="d-flex flex-column flex-lg-row align-items-lg-start justify-content-between gap-3 mb-3">
				<div>
					<h3 class="h4 mb-2">Filter recipes</h3>
				</div>
				<button type="button" class="btn btn-outline-secondary btn-sm" data-clear-filters>Clear filters</button>
			</div>

			<div class="filter-group mb-4">
				<h3 class="h6 text-uppercase text-muted mb-2">Recipe Type</h3>
				<div class="d-flex flex-wrap gap-2">
					<!-- {% for type in site.data.recipe-types.types %}
					<button
						type="button"
						class="btn btn-outline-primary btn-sm filter-chip"
						data-filter-kind="type"
						data-filter-value="{{ type.label | downcase }}"
						aria-pressed="false"
					>
						{{ type.label }}
					</button>
					{% endfor %} -->
				</div>
			</div>

			<div class="filter-group">
				<h3 class="h6 text-uppercase text-muted mb-3">Recipe Tags</h3>
				<div class="d-flex flex-column gap-3">
					<!-- {% for group in site.data.recipe-tags.groups %}
					<div>
						<p class="small text-muted mb-2">{{ group.name }}</p>
						<div class="d-flex flex-wrap gap-2">
							{% for tag in group.tags %}
							<button
								type="button"
								class="btn btn-outline-secondary btn-sm filter-chip"
								data-filter-kind="tag"
								data-filter-value="{{ tag.label | downcase }}"
								aria-pressed="false"
							>
								{{ tag.label }}
							</button>
							{% endfor %}
						</div>
					</div>
					{% endfor %} -->
				</div>
			</div>
		</div>
	</div>

	<div class="row mb-3 g-3 recipes-div" data-recipe-grid>
		{% for recipe in recipes %}
		{% assign recipe_tags = recipe.recipe-tags | default: empty %}
		<div
			class="col-3"
			data-recipe-card
			data-type="{{ recipe.recipe-type | downcase }}"
			data-tags="{% for tag in recipe_tags %}{{ tag | downcase }}{% unless forloop.last %}|{% endunless %}{% endfor %}"
		>
			<a href="{{ recipe.url }}" class="text-decoration-none">
				<div class="card h-100 recipe-card">
					<img src="{{ recipe.thumbnail }}" class="card-img-top" alt="{{ recipe.title }}">
					<div class="card-body d-flex flex-column gap-3">
						<h3 class="h5 card-title mb-2">{{ recipe.title }}</h3>
					</div>
				</div>
			</a>
		</div>
		{% endfor %}
	</div>

	<p class="text-muted d-none" data-empty-state>No recipes match the current filters.</p>
</section>

<script>
	(() => {
		const root = document.querySelector('[data-recipe-catalog]');
		if (!root) {
			return;
		}

		const cards = Array.from(root.querySelectorAll('[data-recipe-card]'));
		const buttons = Array.from(root.querySelectorAll('[data-filter-kind]'));
		const clearButton = root.querySelector('[data-clear-filters]');
		const resultsCount = root.querySelector('[data-results-count]');
		const emptyState = root.querySelector('[data-empty-state]');
		const filterPanel = root.querySelector('[data-filter-panel]');
		const toggleButton = root.querySelector('[data-bs-toggle="collapse"]');

		const normalize = (value) => value.trim().toLowerCase();
		const state = {
			type: '',
			tags: new Set(),
		};

		const readParams = () => {
			const params = new URLSearchParams(window.location.search);
			const type = params.get('type');
			const tags = params.getAll('tag').flatMap((value) => value.split(','));

			state.type = type ? normalize(type) : '';
			state.tags = new Set(tags.filter(Boolean).map(normalize));
		};

		const writeParams = () => {
			const params = new URLSearchParams();
			if (state.type) {
				params.set('type', state.type);
			}
			if (state.tags.size) {
				params.set('tag', Array.from(state.tags).join(','));
			}

			const query = params.toString();
			const nextUrl = query ? `${window.location.pathname}?${query}` : window.location.pathname;
			window.history.replaceState({}, '', nextUrl);
		};

		const syncButtons = () => {
			buttons.forEach((button) => {
				const kind = button.dataset.filterKind;
				const value = button.dataset.filterValue;
				const isActive = kind === 'type' ? state.type === value : state.tags.has(value);

				button.classList.toggle('active', isActive);
				button.setAttribute('aria-pressed', String(isActive));
			});
		};

		const render = () => {
			let visibleCount = 0;

			cards.forEach((card) => {
				const type = card.dataset.type || '';
				const tags = new Set((card.dataset.tags || '').split('|').filter(Boolean));
				const matchesType = !state.type || type === state.type;
				const matchesTags = Array.from(state.tags).every((tag) => tags.has(tag));
				const isVisible = matchesType && matchesTags;

				card.classList.toggle('d-none', !isVisible);
				if (isVisible) {
					visibleCount += 1;
				}
			});

			resultsCount.textContent = `${visibleCount} recipe${visibleCount === 1 ? '' : 's'}`;
			emptyState.classList.toggle('d-none', visibleCount !== 0);
			syncButtons();
			writeParams();
		};

		buttons.forEach((button) => {
			button.addEventListener('click', () => {
				const kind = button.dataset.filterKind;
				const value = button.dataset.filterValue;

				if (kind === 'type') {
					state.type = state.type === value ? '' : value;
				} else if (state.tags.has(value)) {
					state.tags.delete(value);
				} else {
					state.tags.add(value);
				}

				render();
			});
		});

		clearButton.addEventListener('click', () => {
			state.type = '';
			state.tags.clear();
			render();
		});

		filterPanel.addEventListener('shown.bs.collapse', () => {
			toggleButton.textContent = 'Hide filters';
		});

		filterPanel.addEventListener('hidden.bs.collapse', () => {
			toggleButton.textContent = 'Show filters';
		});

		readParams();
		render();

		if (state.type || state.tags.size) {
			const collapse = bootstrap.Collapse.getOrCreateInstance(filterPanel, { toggle: false });
			collapse.show();
			toggleButton.textContent = 'Hide filters';
		}
	})();
</script>