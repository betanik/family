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

.person-with-photo {
    display: flex;
    align-items: center;
    gap: 0.5rem;
}

.person-photo {
    width: 32px;
    height: 32px;
    border-radius: 50%;
    border: 2px solid #3498db;
    object-fit: cover;
    cursor: pointer;
    transition: transform 0.2s;
}

.person-photo:hover {
    transform: scale(1.1);
}

.photo-modal {
    display: none;
    position: fixed;
    z-index: 1000;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0,0,0,0.8);
    animation: fadeIn 0.3s;
}

.photo-modal.show {
    display: flex;
    align-items: center;
    justify-content: center;
}

.photo-modal-content {
    background-color: white;
    padding: 2rem;
    border-radius: 12px;
    max-width: 90%;
    max-height: 90%;
    text-align: center;
    animation: slideUp 0.3s;
}

.photo-modal img {
    max-width: 100%;
    max-height: 70vh;
    border-radius: 8px;
}

.photo-modal-close {
    position: absolute;
    right: 2rem;
    top: 2rem;
    font-size: 2rem;
    cursor: pointer;
    color: white;
}

@keyframes fadeIn {
    from { opacity: 0; }
    to { opacity: 1; }
}

@keyframes slideUp {
    from { transform: translateY(30px); opacity: 0; }
    to { transform: translateY(0); opacity: 1; }
}

@media (max-width: 768px) {
    .photo-modal-content {
        padding: 1rem;
        max-width: 95%;
    }
    
    .person-photo {
        width: 28px;
        height: 28px;
    }
    
    .generation-2 {
        margin: 0.5rem 0 0.5rem 1rem;
        padding: 0.75rem;
    }
    
    .generation-3 {
        margin-left: 1rem;
        font-size: 0.9rem;
    }
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
// Photo mapping for family members
const photoMap = {
  'C1 Lim Ah Bee': '/family/photos/IMG_2612.jpeg',
  'C2 Lim Beng Siang': '/family/photos/IMG_2612.jpeg',
  'C2 Lim Beng Ho': '/family/photos/IMG_2612.jpeg',
  'C7 Lim Beng Soon': '/family/photos/IMG_2612.jpeg',
  'C8 Lim Beng Lee': '/family/photos/IMG_2612.jpeg',
  'C9 Lim Beng Choon': '/family/photos/IMG_2612.jpeg',
  'C10 Lim Beng Hup': '/family/photos/IMG_2612.jpeg',
  'C12 Lim Beng Huat': '/family/photos/IMG_2612.jpeg',
  'C13 Lim Beng Hai': '/family/photos/IMG_2612.jpeg'
};

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
    html += '<div class="tree-node"><span class="toggle">▼</span>';
    html += '<strong class="gender-' + (person.gender === 'F' ? 'f' : 'm') + '">' + person.name + '</strong>';
    if (person.spouse) html += ' & ' + person.spouse;
    html += ' <span class="generation-label">Gen 2</span> <em>(' + childCount + ' children)</em></div>';
    
    html += '<div class="gen2-children">';
    
    // Generation 3
    const gen3Array = data.generation3['line' + person.id] || [];
    gen3Array.forEach(gen3person => {
      const childCount = gen3person.children ? gen3person.children.length : 0;
      const hasPhoto = photoMap[gen3person.name];
      html += '<div class="generation-3">';
      if (childCount > 0) {
        html += '<span class="toggle">▼</span>';
      } else {
        html += '<span class="toggle" style="color: transparent;">–</span>';
      }
      
      if (hasPhoto) {
        html += '<img class="person-photo" src="' + hasPhoto + '" alt="' + gen3person.name + '" title="Click to view" onclick="showPhoto(event)" />';
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
  const html = '<div class="tree-node root">Lim Chey & Repok ak Guem</div><p>Loading full tree data...</p>';
  document.getElementById('familyTree').innerHTML = html;
}

function attachEventListeners() {
  // Use event delegation for better mobile support
  const treeContainer = document.getElementById('familyTree');
  
  if (treeContainer) {
    treeContainer.addEventListener('click', function(event) {
      const toggle = event.target.closest('.toggle');
      if (toggle) {
        event.preventDefault();
        event.stopPropagation();
        toggleChildren(toggle);
      }
    }, true); // Use capture phase for better mobile support
  }
  
  const searchInput = document.getElementById('searchInput');
  if (searchInput) {
    searchInput.addEventListener('keyup', function(e) {
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
}

function toggleChildren(toggleElement) {
  const parent = toggleElement.closest('.generation-2, .generation-3');
  if (!parent) return;
  
  const childrenSelector = toggleElement.closest('.generation-2') ? '.gen2-children' : '.gen3-children';
  const children = parent.querySelector(childrenSelector);
  
  if (children) {
    const isCollapsed = children.classList.contains('collapsed');
    children.classList.toggle('collapsed');
    toggleElement.textContent = isCollapsed ? '▼' : '▶';
  }
}

function showPhoto(event) {
  event.stopPropagation();
  const img = event.target;
  const src = img.src;
  const title = img.alt;
  
  // Create modal if it doesn't exist
  let modal = document.getElementById('photoModal');
  if (!modal) {
    modal = document.createElement('div');
    modal.id = 'photoModal';
    modal.className = 'photo-modal';
    modal.innerHTML = `
      <span class="photo-modal-close" onclick="closePhotoModal(event)">&times;</span>
      <div class="photo-modal-content">
        <h3 id="photoTitle" style="color: #2c3e50; margin-top: 0;"></h3>
        <img id="photoImage" src="" alt="Photo" />
      </div>
    `;
    document.body.appendChild(modal);
    modal.addEventListener('click', function(e) {
      if (e.target === modal) closePhotoModal();
    });
  }
  
  document.getElementById('photoTitle').textContent = title;
  document.getElementById('photoImage').src = src;
  modal.classList.add('show');
}

function closePhotoModal(event) {
  if (event) event.preventDefault();
  const modal = document.getElementById('photoModal');
  if (modal) modal.classList.remove('show');
}
</script>

---

## How to Use This Interactive Tree

- **🔍 Search** - Type any family member's name to filter the tree
- **▼/▶ Expand/Collapse** - Tap/click arrows to show/hide children (works on mobile!)
- **📸 Photos** - Click circular photo thumbnails to view larger photos
- **Color Coding**:
  - 🔴 **Red/Pink** = Female
  - 🔵 **Blue** = Male
  - **Gen 1-4** = Generation level indicator
  
**Mobile Tip**: If expand/collapse isn't working, try tapping the arrow directly

## Tree Structure

1. **Generation 1**: Founder couple (Lim Chey & Repok ak Guem)
2. **Generation 2**: 5 children (A, B, C, D, E with various surnames)
3. **Generation 3**: ~42 grandchildren organized by parent
4. **Generation 4**: ~80+ great-grandchildren (partial listing)

---

<a href="/" class="back-link">← Back to Home</a>
