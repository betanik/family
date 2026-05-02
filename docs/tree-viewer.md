---
layout: default
title: Interactive Family Tree
permalink: /tree-viewer
---

# Interactive Family Tree Viewer

<style>
.tree {
    font-family: 'Courier New', monospace;
    margin: 2rem 0;
    padding: 1.5rem;
    background: white;
    border-radius: 8px;
    box-shadow: 0 2px 10px rgba(0,0,0,0.1);
    line-height: 1.8;
}

.tree-node {
    padding: 0.3rem 0;
}

.tree-node::before {
    content: "├─ ";
    color: #3498db;
    margin-right: 0.5rem;
}

.tree-node.root::before {
    content: "🌳 ";
    margin-right: 0.5rem;
}

.tree-node.root {
    font-weight: bold;
    color: #2c3e50;
    font-size: 1.15rem;
    margin-bottom: 1rem;
    padding: 0.5rem;
    background: linear-gradient(90deg, #e8f4f8 0%, #fff 100%);
    border-radius: 4px;
}

.generation-2 {
    margin: 1rem 0 1rem 1.5rem;
    padding: 1rem;
    background: linear-gradient(90deg, #e3f2fd 0%, #fff 100%);
    border-left: 4px solid #2196f3;
    border-radius: 4px;
}

.generation-3 {
    margin-left: 1.5rem;
    color: #333;
    font-size: 0.95rem;
}

.generation-4 {
    margin-left: 2.5rem;
    color: #666;
    font-size: 0.9rem;
    opacity: 0.9;
}

.toggle {
    cursor: pointer;
    user-select: none;
    color: #2196f3;
    font-weight: bold;
    margin-right: 0.3rem;
    display: inline-block;
    width: 20px;
}

.toggle:hover {
    color: #1976d2;
}

.collapsed {
    display: none !important;
}

.gender-f {
    color: #e91e63;
}

.gender-m {
    color: #1976d2;
}

.search-container {
    margin-bottom: 2rem;
}

.search-container input {
    width: 100%;
    padding: 0.75rem;
    font-size: 1rem;
    border: 2px solid #3498db;
    border-radius: 6px;
    font-family: inherit;
}

.search-container input:focus {
    outline: none;
    border-color: #2196f3;
    box-shadow: 0 0 5px rgba(33, 150, 243, 0.3);
}

.highlight {
    background-color: #fff59d;
    padding: 1px 3px;
    border-radius: 2px;
    font-weight: bold;
}

.generation-label {
    display: inline-block;
    background: #2196f3;
    color: white;
    padding: 0.2rem 0.6rem;
    border-radius: 3px;
    font-size: 0.8rem;
    margin-right: 0.5rem;
    font-weight: bold;
}

.stats {
    background: #f0f4f8;
    padding: 1rem;
    border-radius: 6px;
    margin: 1rem 0;
    font-size: 0.95rem;
}

.stats strong {
    color: #2c3e50;
}
</style>

<div class="search-container">
    <input type="text" id="searchInput" placeholder="🔍 Search family members by name..." />
</div>

<div class="stats">
    <strong>📊 Family Statistics:</strong> 4 Generations | 42+ family members | Last updated: April 13, 2026
</div>

<div class="tree" id="familyTree"></div>

<script>
// Load complete family tree data from JSON
fetch('/family/data/family-tree-complete.json')
  .then(response => response.json())
  .then(data => {
    renderCompleteTree(data);
  })
  .catch(() => {
    console.log('Using embedded data as fallback');
    renderFallbackTree();
  });

function renderCompleteTree(data) {
  let html = '<div class="tree-node root">' + data.generation1.founder.name;
  if (data.generation1.founder.spouse) {
    html += ' & ' + data.generation1.founder.spouse;
  }
  html += ' <span class="generation-label">Gen 1</span></div>';
  
  // Generation 2
  data.generation2.forEach(person => {
    const childCount = person.children ? person.children.length : 0;
    html += '<div class="generation-2">';
    html += '<div class="tree-node"><span class="toggle" onclick="toggleChildren(event)">▼</span>';
    html += '<strong class="gender-' + (person.gender === 'F' ? 'f' : 'm') + '">' + person.name + '</strong>';
    if (person.spouse) html += ' & ' + person.spouse;
    html += ' <span class="generation-label">Gen 2</span> <em>(' + childCount + ' children)</em></div>';
    
    html += '<div class="gen2-children">';
    
    // Generation 3
    const gen3Array = data.generation3['line' + person.id] || [];
    gen3Array.forEach(gen3person => {
      const childCount = gen3person.children ? gen3person.children.length : 0;
      html += '<div class="generation-3">';
      if (childCount > 0) {
        html += '<span class="toggle" onclick="toggleChildren(event)">▼</span>';
      } else {
        html += '<span class="toggle" style="color: transparent;">–</span>';
      }
      html += '<span class="gender-' + (gen3person.gender === 'F' ? 'f' : 'm') + '">' + gen3person.name + '</span>';
      if (gen3person.spouse) html += ' & ' + gen3person.spouse;
      html += ' <span class="generation-label">Gen 3</span>';
      if (gen3person.notes) html += ' <em>(' + gen3person.notes + ')</em>';
      html += '</div>';
      
      // Generation 4
      if (gen3person.children && gen3person.children.length > 0) {
        html += '<div class="gen3-children" style="margin-left: 1rem;">';
        gen3person.children.forEach(gen4id => {
          // Find gen4 person in data
          html += '<div class="generation-4">├─ <span class="gender-m">➤</span> ' + gen4id + '</div>';
        });
        html += '</div>';
      }
    });
    
    html += '</div></div>';
  });
  
  document.getElementById('familyTree').innerHTML = html;
  attachEventListeners();
}

function renderFallbackTree() {
  // Basic fallback rendering
  const html = '<div class="tree-node root">Lim Chey & Repok ak Guem</div><p>Loading full tree data...</p>';
  document.getElementById('familyTree').innerHTML = html;
}

function attachEventListeners() {
  document.querySelectorAll('.toggle').forEach(toggle => {
    toggle.addEventListener('click', toggleChildren);
  });
  
  document.getElementById('searchInput').addEventListener('keyup', function(e) {
    const searchTerm = e.target.value.toLowerCase();
    document.querySelectorAll('.generation-3, .generation-4').forEach(node => {
      if (searchTerm === "") {
        node.style.display = '';
      } else if (node.textContent.toLowerCase().includes(searchTerm)) {
        node.style.display = '';
      } else {
        node.style.display = 'none';
      }
    });
  });
}

function toggleChildren(event) {
  event.preventDefault();
  const toggle = event.target;
  const parent = toggle.closest('.generation-2, .generation-3');
  const children = parent ? parent.querySelector('.gen2-children, .gen3-children') : null;
  
  if (children) {
    children.classList.toggle('collapsed');
    toggle.textContent = children.classList.contains('collapsed') ? '▶' : '▼';
  }
}
</script>

---

## How to Use This Interactive Tree

- **🔍 Search** - Type any family member's name to filter the tree
- **▼/▶ Expand/Collapse** - Click arrows to show/hide children of each generation
- **Color Coding**:
  - 🔴 **Red/Pink** = Female
  - 🔵 **Blue** = Male
  - **Gen 1-4** = Generation level indicator

## Tree Structure

1. **Generation 1**: Founder couple (Lim Chey & Repok ak Guem)
2. **Generation 2**: 5 children (A, B, C, D, E with various surnames)
3. **Generation 3**: ~42 grandchildren organized by parent
4. **Generation 4**: ~80+ great-grandchildren (partial listing)

---

<a href="/" class="back-link">← Back to Home</a>
