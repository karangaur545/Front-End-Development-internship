# Front-End-Development-internship
import React from "react";
import { DndProvider } from "react-dnd";
import { HTML5Backend } from "react-dnd-html5-backend";
import Builder from "./components/Builder";

function App() {
  return (
    <DndProvider backend={HTML5Backend}>
      <Builder />
    </DndProvider>
  );
}

export default App;
import React, { useState } from "react";
import ElementPalette from "./ElementPalette";
import Canvas from "./Canvas";
import PropertyForm from "./PropertyForm";
import "./Builder.css";

export default function Builder() {
  const [elements, setElements] = useState([]);
  const [selectedId, setSelectedId] = useState(null);

  const handleDrop = (type, position) => {
    const id = Date.now().toString();
    setElements([
      ...elements,
      {
        id,
        type,
        position,
        props: getDefaultProps(type)
      }
    ]);
    setSelectedId(id);
  };

  const handleSelect = (id) => setSelectedId(id);

  const handleUpdateProps = (id, newProps) => {
    setElements(elements.map(el => el.id === id ? { ...el, props: newProps } : el));
  };

  return (
    <div className="builder-container">
      <ElementPalette />
      <Canvas
        elements={elements}
        onDrop={handleDrop}
        onSelect={handleSelect}
        selectedId={selectedId}
      />
      <PropertyForm
        element={elements.find(el => el.id === selectedId)}
        onUpdate={handleUpdateProps}
      />
    </div>
  );
}

function getDefaultProps(type) {
  switch (type) {
    case "text":
      return { text: "Sample Text", color: "#333", fontSize: 18 };
    case "image":
      return { src: "https://via.placeholder.com/150", width: 150, height: 100 };
    case "button":
      return { label: "Click Me", bgColor: "#2196f3", color: "#fff" };
    default:
      return {};
  }
}
import React from "react";
import { useDrag } from "react-dnd";
import "./ElementPalette.css";

const PALETTE = [
  { type: "text", label: "Text" },
  { type: "image", label: "Image" },
  { type: "button", label: "Button" }
];

function PaletteItem({ type, label }) {
  const [{ isDragging }, drag] = useDrag({
    type: "ELEMENT",
    item: { type },
    collect: (monitor) => ({
      isDragging: !!monitor.isDragging()
    })
  });
  return (
    <div ref={drag} className={`palette-item ${isDragging ? "dragging" : ""}`}>
      {label}
    </div>
  );
}

export default function ElementPalette() {
  return (
    <div className="palette">
      <h3>Elements</h3>
      {PALETTE.map(item => (
        <PaletteItem key={item.type} {...item} />
      ))}
    </div>
  );
}
import React from "react";
import { useDrop } from "react-dnd";
import "./Canvas.css";

export default function Canvas({ elements, onDrop, onSelect, selectedId }) {
  const [{ isOver }, drop] = useDrop({
    accept: "ELEMENT",
    drop: (item, monitor) => {
      const offset = monitor.getSourceClientOffset();
      onDrop(item.type, { x: offset.x - 250, y: offset.y - 60 }); // adjust for palette width
    },
    collect: monitor => ({
      isOver: !!monitor.isOver()
    })
  });

  return (
    <div ref={drop} className={`canvas ${isOver ? "over" : ""}`}>
      {elements.map(el => (
        <ElementPreview
          key={el.id}
          element={el}
          selected={el.id === selectedId}
          onClick={() => onSelect(el.id)}
        />
      ))}
    </div>
  );
}

function ElementPreview({ element, selected, onClick }) {
  const style = {
    position: "absolute",
    left: element.position.x,
    top: element.position.y,
    border: selected ? "2px solid #2196f3" : "1px solid #ccc",
    padding: 4,
    cursor: "pointer",
    background: "#fff"
  };
  switch (element.type) {
    case "text":
      return (
        <div style={style} onClick={onClick}>
          <span style={{ color: element.props.color, fontSize: element.props.fontSize }}>
            {element.props.text}
          </span>
        </div>
      );
    case "image":
      return (
        <div style={style} onClick={onClick}>
          <img
            src={element.props.src}
            width={element.props.width}
            height={element.props.height}
            alt=""
          />
        </div>
      );
    case "button":
      return (
        <div style={style} onClick={onClick}>
          <button
            style={{
              background: element.props.bgColor,
              color: element.props.color,
              border: "none",
              padding: "8px 16px",
              borderRadius: 4
            }}
          >
            {element.props.label}
          </button>
        </div>
      );
    default:
      return null;
  }
}
import React from "react";
import "./PropertyForm.css";

export default function PropertyForm({ element, onUpdate }) {
  if (!element) return <div className="property-form">Select an element</div>;

  const handleChange = (e) => {
    const { name, value } = e.target;
    onUpdate(element.id, { ...element.props, [name]: value });
  };

  const { type, props } = element;

  return (
    <div className="property-form">
      <h3>Properties</h3>
      {type === "text" && (
        <>
          <label>Text</label>
          <input name="text" value={props.text} onChange={handleChange} />
          <label>Color</label>
          <input name="color" type="color" value={props.color} onChange={handleChange} />
          <label>Font Size</label>
          <input name="fontSize" type="number" value={props.fontSize} onChange={handleChange} />
        </>
      )}
      {type === "image" && (
        <>
          <label>Image URL</label>
          <input name="src" value={props.src} onChange={handleChange} />
          <label>Width</label>
          <input name="width" type="number" value={props.width} onChange={handleChange} />
          <label>Height</label>
          <input name="height" type="number" value={props.height} onChange={handleChange} />
        </>
      )}
      {type === "button" && (
        <>
          <label>Label</label>
          <input name="label" value={props.label} onChange={handleChange} />
          <label>Background Color</label>
          <input name="bgColor" type="color" value={props.bgColor} onChange={handleChange} />
          <label>Text Color</label>
          <input name="color" type="color" value={props.color} onChange={handleChange} />
        </>
      )}
    </div>
  );
}
.builder-container {
  display: flex;
  flex-direction: row;
  height: 100vh;
}
@media (max-width: 800px) {
  .builder-container {
    flex-direction: column;
  }
}
.palette {
  width: 200px;
  background: #f5f5f5;
  padding: 16px;
  border-right: 1px solid #ddd;
}
.palette-item {
  background: #fff;
  border: 1px solid #bbb;
  margin: 8px 0;
  padding: 12px;
  cursor: grab;
  border-radius: 4px;
  text-align: center;
  transition: background 0.2s;
}
.palette-item.dragging {
  background: #e0e0e0;
}
.canvas {
  flex: 1;
  position: relative;
  background: #fafafa;
  margin: 0 16px;
  border: 1px solid #eee;
  min-height: 400px;
  overflow: auto;
}
.canvas.over {
  background: #e3f2fd;
}
.property-form {
  width: 250px;
  background: #f5f5f5;
  padding: 16px;
  border-left: 1px solid #ddd;
}
.property-form label {
  display: block;
  margin-top: 12px;
  font-size: 14px;
}
.property-form input {
  width: 100%;
  margin-top: 4px;
  padding: 6px;
  border: 1px solid #ccc;
  border-radius: 3px;
}
