---
layout: default
title: Interactive Family Tree
permalink: /tree-viewer
---

# Interactive Family Tree Viewer

<style>
.tree {
    font-family: monospace;
    margin: 2rem 0;
    padding: 1rem;
    background: white;
    border-radius: 8px;
    box-shadow: 0 2px 10px rgba(0,0,0,0.1);
}

.tree-node {
    margin-left: 2rem;
    padding: 0.5rem 0;
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
    margin-left: 0;
    font-weight: bold;
    color: #2c3e50;
    font-size: 1.1rem;
}

.generation-2 {
    margin-left: 2rem;
    padding: 0.75rem;
    background: linear-gradient(90deg, #e8f4f8 0%, #fff 100%);
    border-left: 4px solid #3498db;
    margin: 1rem 0 1rem 2rem;
    border-radius: 4px;
}

.generation-3 {
    margin-left: 2rem;
    color: #555;
}

.toggle {
    cursor: pointer;
    user-select: none;
    color: #3498db;
    font-weight: bold;
    margin-right: 0.5rem;
}

.toggle:hover {
    color: #2980b9;
}

.collapsed {
    display: none;
}

.gender-f {
    color: #e91e63;
}

.gender-m {
    color: #2196f3;
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
}

.highlight {
    background-color: #fff59d;
    padding: 2px 4px;
    border-radius: 2px;
}
</style>

<div class="search-container">
    <input type="text" id="searchInput" placeholder="🔍 Search family members by name..." />
</div>

<div class="tree" id="familyTree"></div>

<script>
// Load and parse family tree data
const familyData = {
  "founder": {
    "name": "Lim Chey (Haitang, Amor, China)",
    "spouse": "Repok ak Guem (Seburau)",
    "generation": 1
  },
  "children": [
    {
      "id": "A",
      "name": "A Swee Guan",
      "spouse": "Chandang Ak Ersan",
      "generation": 2,
      "children": [
        { "name": "A1 Teck Dee", "gender": "M" },
        { "name": "A2 Choo Lan", "gender": "F" },
        { "name": "A3 Choo Hoi", "gender": "F" },
        { "name": "A4 Teck Soon", "gender": "M" },
        { "name": "A5 Juana", "gender": "F" }
      ]
    },
    {
      "id": "B",
      "name": "B Lim Swee Chai",
      "generation": 2,
      "children": [
        { "name": "B1 Teck Seng", "gender": "M" },
        { "name": "B2 Teck Hin", "gender": "M" },
        { "name": "B3 Teck Hong", "gender": "M" },
        { "name": "B4 Goh Tiam", "gender": "M" },
        { "name": "B5 Ah Nong", "gender": "M" },
        { "name": "B6 Teck Eng", "gender": "F" },
        { "name": "B7 Teck Neo", "gender": "F" }
      ]
    },
    {
      "id": "C",
      "name": "C Lim Swee Kim",
      "spouse": "Chin Nyong Jin",
      "generation": 2,
      "children": [
        { "name": "C1 Ah Bee", "gender": "M" },
        { "name": "C2 Beng Siang", "gender": "M" },
        { "name": "C3 Beng Ho", "gender": "M" },
        { "name": "C4 Nong Chik", "gender": "F" },
        { "name": "C5 Ah Tin", "gender": "F" },
        { "name": "C6 Mary", "gender": "F" },
        { "name": "C7 Beng Soon", "gender": "M" },
        { "name": "C8 Beng Lee", "gender": "M" },
        { "name": "C9 Beng Choon", "gender": "M" },
        { "name": "C10 Beng Hup", "gender": "M" },
        { "name": "C11 Lucy (died at birth)", "gender": "F" },
        { "name": "C12 Beng Huat", "gender": "M" },
        { "name": "C13 Beng Hai", "gender": "M" }
      ]
    },
    {
      "id": "D",
      "name": "D Lim Swee Eng",
      "spouse": "Heng Soen",
      "generation": 2,
      "children": [
        { "name": "D1 Ea Arin", "gender": "M" }
      ]
    },
    {
      "id": "E",
      "name": "E Lim Swee Hock",
      "spouse": "Teo Ah Kim",
      "generation": 2,
      "children": [
        { "name": "E1 Teo Ah Bee", "gender": "F" },
        { "name": "E2 Helen Teo", "gender": "F" },
        { "name": "E3 Teo Hong Tai", "gender": "F" },
        { "name": "E4 Chua Khai Seng", "gender": "M" },
        { "name": "E5 Chua Kui Choon", "gender": "M" }
      ]
    }
  ]
};

function renderTree(data, searchTerm = "") {
  let html = '<div class="tree-node root">' + data.founder.name;
  if (data.founder.spouse) {
    html += ' & ' + data.founder.spouse;
  }
  html += ' (Generation 1)</div>';
  
  data.children.forEach(child => {
    html += '<div class="generation-2">';
    html += '<div class="tree-node"><span class="toggle" onclick="toggleChildren(this)">▼</span>';
    html += '<strong>' + child.name + '</strong>';
    if (child.spouse) html += ' & ' + child.spouse;
    html += ' <em>(' + child.children.length + ' children)</em></div>';
    
    html += '<div class="children">';
    child.children.forEach(kid => {
      const genderClass = kid.gender === 'F' ? 'gender-f' : 'gender-m';
      html += '<div class="generation-3 ' + genderClass + '">';
      html += '├─ ' + kid.name;
      if (kid.gender) html += ' (' + kid.gender + ')';
      html += '</div>';
    });
    html += '</div></div>';
  });
  
  return html;
}

document.getElementById('familyTree').innerHTML = renderTree(familyData);

function toggleChildren(element) {
  const parent = element.closest('.generation-2');
  const children = parent.querySelector('.children');
  children.classList.toggle('collapsed');
  element.textContent = children.classList.contains('collapsed') ? '▶' : '▼';
}

// Search functionality
document.getElementById('searchInput').addEventListener('keyup', function(e) {
  const searchTerm = e.target.value.toLowerCase();
  const nodes = document.querySelectorAll('.generation-3');
  
  nodes.forEach(node => {
    if (searchTerm === "") {
      node.style.display = '';
      node.innerHTML = node.innerHTML.replace(/<span class="highlight">/g, '').replace(/<\/span>/g, '');
    } else if (node.textContent.toLowerCase().includes(searchTerm)) {
      node.style.display = '';
      node.innerHTML = node.innerHTML.replace(/<span class="highlight">/g, '').replace(/<\/span>/g, '');
      node.innerHTML = node.innerHTML.replace(searchTerm, '<span class="highlight">' + searchTerm + '</span>');
    } else {
      node.style.display = 'none';
    }
  });
});
</script>

---

## How to Use

- **Click the arrows (▼/▶)** next to generation 2 members to expand/collapse their children
- **Search** by typing a family member's name in the search box
- **Color coding**: 🔵 Blue = Male, 🔴 Red/Pink = Female

<a href="/" class="back-link">← Back to Home</a>
