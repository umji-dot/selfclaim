# Design System Specification
## 셀프클레임 v3 - Design System Document

**Project**: 셀프클레임 (Self-Claim) v3  
**Version**: 3.0  
**Last Updated**: 2026년 6월 18일  
**Purpose**: Enterprise Field Service Platform - Mobile First

---

## 📋 목차
1. [디자인 토큰](#디자인-토큰)
2. [컴포넌트 시스템](#컴포넌트-시스템)
3. [레이아웃 시스템](#레이아웃-시스템)
4. [디자인 원칙](#디자인-원칙)

---

## 디자인 토큰

### Color Palette

#### Grayscale (Neutral Colors)
```
--black:   #333333      (Primary text, headings)
--gray-1:  #525252      (Secondary text)
--gray-2:  #777777      (Tertiary text)
--gray-3:  #B7B7B7      (Disabled state, borders)
--gray-4:  #DDDDDD      (Light borders, dividers)
--gray-5:  #E6E6E6      (Very light background)
--gray-6:  #F7F7F7      (Light background, surface)
--white:   #FFFFFF      (Pure white, primary background)
```

#### Primary Color
**Brand**: DB손해보험 (DB Insurance)
```
--primary:        #008449 (100% - Default, Primary Actions)
--primary-point:  rgba(0, 132, 73, 0.80) (80% - Hover state)
--primary-box:    rgba(0, 132, 73, 0.08) (8% - Light background)
--primary-light:  rgba(0, 132, 73, 0.02) (2% - Very light background)
```

#### Semantic Colors (Status)
```
--success: #3CC456    (Success state, positive action, confirmation)
--error:   #FF5555    (Error state, destructive action, alert)
```

#### Secondary Colors (Process Flow - from Figma)
- **Purple**: Decision diamonds, primary flow nodes
- **Light Green**: Input/process steps, information
- **Teal/Cyan**: Alternative process steps
- **Gray**: Secondary actions, neutral states

#### AI/Chat Interface Colors
```
--ai-surface:         #F4F4F6   (Chat background)
--ai-line:            #E3EBE6   (Dividers in chat)
--ai-muted:           #F0F4F2   (Muted UI elements)
--ai-muted-strong:    #E6EEE9   (Strong muted elements)
--ai-user:            #EAF7F0   (User input background)
```

---

### Typography Scale

#### Font Family
**Primary**: "Pretendard Variable"
**Fallback**: -apple-system, BlinkMacSystemFont, "Apple SD Gothic Neo", "Noto Sans KR", sans-serif

#### Font Sizes & Weights

| Style | Size | Line Height | Weight | Usage |
|-------|------|-------------|--------|-------|
| **Title 1** | 38px | 46px | 700 | Major page titles, hero sections |
| **Title 2** | 30px | 42px | 700 | Section headers, modal titles |
| **Title 3** | 24px | 24px | 700 | Primary color (Green) | Subsection titles |
| **Headline 1** | 20px | 30px | 700 | Card titles, important labels |
| **Headline 2** | 18px | 24px | 800 | Emphasis text, strong labels |
| **Headline 3** | 16px | 24px | 700 | Navigation labels, button text |
| **Body 1** | 15px | 22px | 400 | Primary body text, descriptions |
| **Body 2** | 14px | 20px | 400 | Secondary body text, lists |
| **Body 3** | 13px | 18px | 400 | Tertiary body text, fine print |
| **Caption 1** | 12px | 15px | 500 | Helper text, metadata labels |

#### Letter Spacing
```
Global: -0.01em (for tight visual harmony)
Korean text: 0em (default for improved readability)
```

---

### Spacing Scale

#### Base Unit: 4px
```
4px   (xs)  - Minimal spacing between adjacent elements
8px   (sm)  - Small components, internal padding
12px  (md)  - Standard component spacing
16px  (lg)  - Major section spacing
20px  (xl)  - Large gaps between major sections
24px  (2xl) - Extra large spacing for breathing room
```

#### Common Spacing Values
- **Padding**: 
  - Buttons: 12-16px (horizontal), 10-14px (vertical)
  - Input: 12-14px (horizontal), 10-12px (vertical)
  - Card: 16-18px
  - Modal: 20-24px

- **Margin**:
  - Between sections: 16-24px
  - Between components: 8-12px
  - Between elements in list: 8px

---

### Border Radius

```
--radius-0:    0px     (No rounding - rare)
--radius-sm:   4px     (Small elements, badges)
--radius-md:   8px     (Subtle rounding, dividers)
--radius-lg:   12px    (Cards, modals, dropdowns)
--radius-xl:   14-16px (Buttons, input fields)
--radius-2xl:  20px    (Large cards, bottom sheets)
--radius-full: 999px   (Pills, avatars, chips)
```

#### Application Examples
- **Buttons**: 12px
- **Input Fields**: 12px
- **Cards**: 14px
- **Modal Dialogs**: 14px
- **Badges/Chips**: 10px (smaller), 20px (larger)
- **Bottom Sheet**: 20px (top only)
- **Avatars**: 50% (circular)

---

### Shadow

#### Elevation Levels

```
--shadow-xs:  0 1px 2px rgba(0, 0, 0, 0.05)
              Small shadows for subtle elevation

--shadow-sm:  0 2px 6px rgba(0, 0, 0, 0.12)
              Default shadow for cards, dropdowns

--shadow-md:  0 4px 12px rgba(0, 0, 0, 0.15)
              Medium elevation for important cards

--shadow-lg:  0 8px 24px rgba(0, 0, 0, 0.20)
              Large shadow for modals, popovers

--shadow-xl:  0 12px 32px rgba(0, 0, 0, 0.25)
              Extra large for floating actions
```

#### Usage Guidelines
- **No Shadow**: Flat design, backgrounds, borders
- **XS/SM Shadow**: Input fields, small cards, message bubbles
- **SM Shadow**: Default for cards, chips, buttons
- **MD/LG Shadow**: Modals, overlays, dropdowns
- **LG/XL Shadow**: Floating buttons, critical overlays

---

### Transition & Animation

```
--ease-linear:      linear
--ease-ease-in:     cubic-bezier(0.4, 0, 1, 1)
--ease-ease-out:    cubic-bezier(0, 0, 0.2, 1)
--ease-ease-in-out: cubic-bezier(0.4, 0, 0.2, 1)

--duration-fast:    150ms  (UI interactions, hover states)
--duration-normal:  300ms  (Standard transitions)
--duration-slow:    500ms  (Page transitions, animations)
```

**Default Transition**: 
```css
transition: all 0.2s ease;
```

---

## 컴포넌트 시스템

### Button

#### Purpose
Primary call-to-action element for user interactions. Used for form submissions, navigation, and important actions.

#### Variants

##### Style
- **Primary** (Filled): Full background with primary color (#008449)
- **Secondary** (Outline): Bordered with primary color
- **Tertiary** (Text): Text-only, minimal styling
- **Danger** (Destructive): Red background (#FF5555)
- **Success** (Positive): Green background (#3CC456)
- **Disabled**: Reduced opacity (0.35), no interaction

##### Size
- **Small (sm)**: 36-40px height, 12px font, 12px padding
- **Medium (md)**: 44-48px height, 14-15px font, 16px padding
- **Large (lg)**: 52-56px height, 16px font, 20px padding

##### State
- **Default**: Standard appearance
- **Hover**: opacity +10%, shadow +1 level
- **Active/Pressed**: scale(0.97), darker background
- **Focus**: outline or glow effect
- **Disabled**: opacity 0.35, no cursor changes

#### Usage Rules
- Primary action: Use Primary style, Medium size
- Secondary action: Use Secondary or Tertiary style
- Destructive action: Use Danger variant (e.g., Delete)
- Confirmation: Use Success variant for positive confirmation
- Disabled state: Show clear visual difference
- Mobile: Minimum 44px height for touch targets
- Full-width buttons: Used in forms, modals, bottom sheets

#### Implementation
```html
<!-- Primary Button -->
<button class="btn btn-primary btn-md">
  {{ label }}
</button>

<!-- Secondary Button -->
<button class="btn btn-secondary btn-md">
  {{ label }}
</button>

<!-- Danger Button -->
<button class="btn btn-danger btn-md">
  Delete
</button>

<!-- Disabled State -->
<button class="btn btn-primary btn-md" disabled>
  {{ label }}
</button>
```

---

### Input Fields

#### Text Input (input-text)

**Purpose**: Text entry fields for user input (names, descriptions, etc.)

**Size**:
- Height: 44-48px
- Padding: 12-14px (horizontal), 10-12px (vertical)
- Border radius: 12px

**State**:
- Default: Gray border (#DDDDDD), white background
- Focus: Primary color border (#008449), shadow-xs
- Filled: Same as default
- Error: Red border (#FF5555), optional error text below
- Disabled: Gray background (#F7F7F7), gray text (#B7B7B7)

**Placeholder**: Gray text (#BBB), font-size 15px

---

#### Select/Dropdown (input-select)

**Purpose**: Dropdown selection for predefined options

**Components**:
- Container: 44-48px height, 12px border-radius
- Arrow icon: 20x20px, positioned right
- Options: Full-width overlay, smooth scroll

**State**:
- Closed: Shows selected value
- Open: Shows all options, highlight on hover
- Error: Red border
- Disabled: Gray background, no interaction

---

#### Chat/Message Input (Chat input)

**Purpose**: Specialized input for chat/messaging interfaces

**Design**:
- Height: 48px
- Multi-line capable (auto-expand)
- Action button (Send) integrated on right
- Placeholder text for guidance

**Features**:
- Auto-expand when scrollable
- Send button integration
- Message validation
- Debounced input handling

---

### Message Components

#### Message Bubble (MessageBubble)

**Purpose**: Display chat messages or notifications

**Variants**:
- **Bot Message**: White background, left-aligned, border-radius 16px 16px 16px 4px
- **User Message**: Primary color background (#008449), white text, right-aligned
- **System Message**: Gray background, centered

**Styling**:
- Max-width: 80% of container
- Padding: 12-16px
- Shadow: 0 1px 2px rgba(0,0,0,0.05)
- Line-height: 1.65

**Typography**:
- Default: Body 2 (14px, 400)
- Bold text: Headline 3 weight (700)
- Emphasis: Primary color highlight

---

#### Header Component

**Purpose**: Top navigation bar with app identity and controls

**Layout**:
- Height: 52-72px
- Sticky positioning (top: 0, z-index: 10)
- Padding: 14px 18-24px
- Border-bottom: 1px solid transparent (changes on scroll)

**Contents**:
- Logo/Brand name (left)
- Title (center/left)
- Action buttons (right)
- Status indicators

---

#### Composer Component

**Purpose**: Message composition area with input and send

**Layout**:
- Container: 48px height
- Input field: Flexible width, 14px font
- Send button: Fixed width, 40px

**State**:
- Empty: Send button disabled (opacity 0.35)
- Has text: Send button active
- Sending: Button shows loading state

---

### Icons & Symbols

#### Included Icons
- **keyboard_double_arrow_right**: Navigation arrow
- **keyboard_arrow_down**: Dropdown indicator
- **close**: Dismiss/cancel action
- **send**: Message submission
- **User-btn**: User profile button

#### Style
- Size: 20x20px (default), 24x24px (large), 16x16px (small)
- Color: Inherit from parent (icon-only components are monochrome)
- Stroke: 2px for 20px icons, 1.5px for others

---

### Control Elements

#### Chip Component

**Purpose**: Small interactive label/tag elements

**Design**:
- Height: 32px
- Padding: 6-10px horizontal
- Border-radius: 20px (pill-shaped)
- Border: 1px or solid color

**Variants**:
- **Filled**: Colored background, white text
- **Outlined**: Transparent background, colored border
- **Avatar**: Icon + text format

**State**:
- Default: Standard appearance
- Selected: Highlight or checkmark
- Disabled: Reduced opacity

**Usage**: Filtering, tagging, quick actions

---

#### Bottom Sheet

**Purpose**: Modal drawer from bottom of screen

**Design**:
- Border-radius: 20px (top only)
- Height: Dynamic, max 80% viewport
- Background: White (#FFFFFF)
- Shadow: 0 -4px 12px rgba(0,0,0,0.15)

**Structure**:
- Drag handle (top)
- Header area
- Content area (scrollable)
- Optional footer with actions

**Animation**:
- Slide-up from bottom
- Duration: 300ms
- Easing: ease-out

---

### Data Display

#### Table Structure (Implied from Components)

**Anticipated table components**:
- Header row: Bold text, border-bottom
- Data rows: Regular weight, borders between rows
- Hover state: Slight background color change
- Selection: Checkbox integration

---

## 레이아웃 시스템

### Grid System

#### Base Unit
- **Grid**: 4px base unit
- **Column count**: 12-column responsive grid

#### Breakpoints & Device Widths
```
Mobile:  max-width: 430px    (Primary - Main target)
Tablet:  max-width: 768px    (Secondary)
Desktop: max-width: 1200px   (Tertiary)
```

#### Container Widths
```
--container-mobile:   420px   (with 5px safe margin)
--container-tablet:   750px
--container-desktop:  1160px
```

### Safe Area / Viewport

#### Mobile Safe Area Consideration
```css
padding: max(0, env(safe-area-inset-top, 0)) 
         max(0, env(safe-area-inset-right, 0)) 
         max(0, env(safe-area-inset-bottom, 0)) 
         max(0, env(safe-area-inset-left, 0));
```

#### Vertical Spacing
- Top padding: 52-72px (header height)
- Bottom padding: 16-24px (or safe-area-inset-bottom)
- Content padding: 16px (mobile), 20-24px (tablet+)

### Page Structure

#### Mobile Layout Pattern
```
┌─────────────────┐
│     Header      │ 52-72px (sticky)
├─────────────────┤
│               │
│   Content     │ Main scrollable area
│   Area        │
│               │
├─────────────────┤
│  Bottom        │ Optional floating/fixed
│  Controls      │ area
└─────────────────┘
```

#### Responsive Behavior
- **Mobile (< 430px)**: Single column, full-width
- **Tablet (430px - 768px)**: 2 columns, side-by-side possible
- **Desktop (> 768px)**: Multi-column, complex layouts possible

### Content Width
- **Mobile**: 100% - 32px (16px padding each side)
- **Tablet**: 100% - 40px
- **Desktop**: max(90%, 1160px)

---

## 디자인 원칙

### 1. **Enterprise + Mobile First**
- Designed for business users in field service environments
- Prioritizes mobile usability (small screens, one-handed use)
- Touch-friendly interaction targets (minimum 44x44px)
- Supports offline functionality concept

### 2. **Clarity & Efficiency**
- Information hierarchy prioritizes most important content
- Minimal cognitive load through clear typography
- Direct navigation without unnecessary steps
- Fast visual scanning with consistent patterns

### 3. **Trust & Professionalism**
- Corporate identity (DB Insurance brand green)
- Clean, organized visual structure
- Consistent spacing and alignment
- Professional color palette (green, neutral grays)

### 4. **Accessibility**
- Sufficient color contrast (WCAG AA standard)
- Clear focus states for keyboard navigation
- Readable font sizes (minimum 14px for body text)
- Semantic HTML structure
- Alternative text for icons

### 5. **Consistency**
- Unified component library across app
- Predictable interaction patterns
- Consistent icon style and sizing
- Standardized spacing grid (4px base unit)

### 6. **Responsiveness**
- Graceful scaling from mobile to desktop
- Touch-optimized for mobile-first approach
- Readable typography at all sizes
- Flexible layouts that adapt to content

### 7. **Speed & Performance**
- Minimal visual complexity
- Fast, smooth animations (max 300ms)
- Optimized for mobile networks
- Progressive enhancement strategy

### 8. **Data-Heavy Dashboard Ready**
- Supports complex information display
- Clear visual hierarchy for nested data
- Efficient space utilization
- Pattern repetition for scanning

---

## 구현 고려사항

### Performance
- CSS variables for theming
- Minimal JavaScript dependencies
- Hardware acceleration for animations
- Responsive image loading

### Browser Support
- Modern browsers (Chrome, Safari, Firefox)
- iOS Safari 12+ (mobile focus)
- Android Chrome 80+
- Progressive enhancement for older browsers

### Dark Mode (Future)
- CSS variable-based theming ready
- Current spec: Light theme only
- Dark theme tokens to be added in v3.1

### Internationalization
- Korean as primary language
- Font supports full Korean character set
- Flexible spacing for text expansion
- RTL support to be added if needed

---

## 변경 기록

| Version | Date | Changes |
|---------|------|---------|
| 3.0 | 2026-06-18 | Initial Design System Documentation |

---

## 추가 리소스

### Design Files
- Figma: `셀프클레임 v3` project
- Pages: Page 1 (Overview), Page 2 (Component Flow)

### Implementation Resources
- HTML/CSS Tokens: [a.html](a.html), [b.html](b.html), [c.html](c.html)
- Component Library: Defined in Figma components
- Color Variables: `:root` CSS variables in main stylesheet

---

**Document Owner**: UX Design Team  
**Last Review**: 2026-06-18  
**Status**: Active (v3.0)

---

## Appendix: Color Reference Card

### Quick Color Guide
```
Primary Actions:     #008449 (Green)
Text:               #333333 (Black)
Secondary Text:     #525252 (Dark Gray)
Borders:            #DDDDDD (Light Gray)
Backgrounds:        #F7F7F7 (Very Light Gray)
Error/Destructive:  #FF5555 (Red)
Success:            #3CC456 (Green)
```

### Component Color Mapping
| Component | Color | Hex Code |
|-----------|-------|----------|
| Primary Button | Primary | #008449 |
| Success State | Success | #3CC456 |
| Error State | Error | #FF5555 |
| Disabled | Gray 3 | #B7B7B7 |
| Background | White/Gray-6 | #FFFFFF / #F7F7F7 |
| Text | Black | #333333 |
| Borders | Gray-4 | #DDDDDD |

