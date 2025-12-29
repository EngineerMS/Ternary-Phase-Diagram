# Ternary-Phase-Diagram
You can use it to obtain inorganic solution phase diagrams. 
import matplotlib.pyplot as plt
import numpy as np

def ternary_to_cartesian(a, b, c):
    """
    Converts ternary coordinates (A, B, C) to Cartesian (x, y).
    """
    total = a + b + c
    a = a / total
    b = b / total
    c = c / total
    
    x = b + c/2
    y = (np.sqrt(3) / 2) * c
    return x, y

# --- DATA INPUT (25°C Data) ---
data_points = [
    (12.6, 16.5, "S.10"),
    (21.2, 14.8, "S.10 + S"),
    (24.9, 14.6, "S"),
    (25.9, 13.9, "S + 1.1.2"), # Eutectic point
    (26.8, 13.3, "1.1.2"),
    (27.7, 12.7, "1.1.2"), 
    (29.5, 11.3, "1.1.2"),
    (31.1, 10.2, "1.1.2"),
    (37.7, 6.17, "1.1.2")
]

na_no3 = np.array([p[0] for p in data_points])
na_2so4 = np.array([p[1] for p in data_points])
water = 100 - (na_no3 + na_2so4)
labels = [p[2] for p in data_points]

x_coords, y_coords = ternary_to_cartesian(na_no3, na_2so4, water)

# --- PLOTTING ---
# Increased figure height slightly to give more room for the header
fig, ax = plt.subplots(figsize=(12, 12)) 

# 1. Main Triangle Frame
tri_x = [0, 1, 0.5, 0]
tri_y = [0, 0, np.sqrt(3)/2, 0]
ax.plot(tri_x, tri_y, 'k-', linewidth=2, zorder=1)

# 2. GRID & AXIS LABELS
ticks = np.linspace(0, 1, 11)

for t in ticks:
    if t == 0 or t == 1: continue
    
    # Grid Lines
    x1, y1 = ternary_to_cartesian(1-t, 0, t)
    x2, y2 = ternary_to_cartesian(0, 1-t, t)
    ax.plot([x1, x2], [y1, y2], 'k--', alpha=0.15, linewidth=1, zorder=0) # Horizontal
    
    x1, y1 = ternary_to_cartesian(1-t, t, 0)
    x2, y2 = ternary_to_cartesian(0, t, 1-t)
    ax.plot([x1, x2], [y1, y2], 'k--', alpha=0.15, linewidth=1, zorder=0) # Left-Right

    x1, y1 = ternary_to_cartesian(t, 1-t, 0)
    x2, y2 = ternary_to_cartesian(t, 0, 1-t)
    ax.plot([x1, x2], [y1, y2], 'k--', alpha=0.15, linewidth=1, zorder=0) # Right-Left

    # Axis Percentage Labels
    lx, ly = ternary_to_cartesian(1-t, 0, t)
    ax.text(lx - 0.02, ly, f'{int(t*100)}', fontsize=8, color='blue', ha='right', va='center')
    
    bx, by = ternary_to_cartesian(1-t, t, 0)
    ax.text(bx, by - 0.03, f'{int(t*100)}', fontsize=8, color='green', ha='center', va='top')
    
    rx, ry = ternary_to_cartesian(t, 0, 1-t)
    ax.text(rx + 0.02, ry, f'{int(t*100)}', fontsize=8, color='red', ha='left', va='center')

# 3. DATA PLOTTING
ax.plot(x_coords, y_coords, 'b-', linewidth=1.5, alpha=0.6, label='25°C Solubility Curve', zorder=4)
ax.scatter(x_coords, y_coords, s=80, c='red', edgecolors='black', label='Experimental Data', zorder=5)

# 4. SMART LABELS (Prevent Overlap)
for i, txt in enumerate(labels):
    x, y = x_coords[i], y_coords[i]
    
    # Custom offsets for specific crowded points
    xytext = (15, 10) 
    ha = 'left'
    connection_style = "arc3,rad=0.2"
    
    if "S.10" in txt and "S" not in txt: # First point (Glauber salt)
        xytext = (20, 10)
    elif "S.10 + S" in txt: # Transition
        xytext = (30, -5)
    elif txt == "S": # Anhydrous
        xytext = (-25, 15)
        ha = 'right'
    elif "S + 1.1.2" in txt: # Invariant Point (CRITICAL)
        xytext = (40, -30) # Move far down-right
        txt = "Invariant Point\n(S + 1.1.2)"
        connection_style = "arc3,rad=-0.3"
    elif i > 4 and i < len(labels)-1: 
        continue # Skip intermediate points to clean up
        
    if i == len(labels)-1: # Last point label
        txt = "1.1.2 (Darapskite)"
        xytext = (10, -20)

    ax.annotate(txt, 
                 (x, y),
                 xytext=xytext, textcoords='offset points',
                 fontsize=9, weight='bold', ha=ha,
                 arrowprops=dict(arrowstyle="->", connectionstyle=connection_style, color='black'),
                 zorder=6)

# 5. CORNER LABELS (Positioned closer to vertices)
ax.text(-0.05, -0.08, 'NaNO3 (100%)\n(Red Axis)', fontsize=12, weight='bold', ha='center', color='red')
ax.text(1.05, -0.08, 'Na2SO4 (100%)\n(Green Axis)', fontsize=12, weight='bold', ha='center', color='green')
# Moved WATER label slightly closer to the tip (0.06 instead of 0.1) to make room for title
ax.text(0.5, np.sqrt(3)/2 + 0.06, 'WATER (100%)\n(Blue Axis)', fontsize=12, weight='bold', ha='center', color='blue')

# --- LAYOUT FIX (THE SOLUTION) ---
# Turn off axes box
ax.axis('off')

# Push the entire plot area DOWN to leave 20% empty space at the top for the title
plt.subplots_adjust(top=0.80, bottom=0.1, left=0.1, right=0.9)

# Place the title at the very top of the FIGURE (not the axes)
plt.suptitle('NaNO3 - Na2SO4 - H2O System (25°C)\nTernary Phase Diagram', 
             fontsize=18, weight='bold', y=0.95)

# Place legend outside
plt.legend(loc='upper right', bbox_to_anchor=(1.05, 0.9))

plt.show()
