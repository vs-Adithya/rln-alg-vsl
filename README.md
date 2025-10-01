<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<title>Relational Algebra Visualizer</title>
<style>
  @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;600&display=swap');
  body {
    font-family: 'Inter', sans-serif;
    background: #f7f9fc;
    margin: 20px;
    color: #2c3e50;
  }
  h1, h2 {
    text-align: center;
    font-weight: 600;
    color: #34495e;
  }
  .container {
    max-width: 900px;
    margin: 0 auto 40px auto;
    background: white;
    padding: 20px 30px;
    border-radius: 12px;
    box-shadow: 0 8px 20px rgb(0 0 0 / 0.1);
  }
  label {
    font-weight: 600;
    display: block;
    margin-bottom: 6px;
  }
  input[type=text], select, textarea {
    width: 100%;
    padding: 8px 10px;
    margin-bottom: 12px;
    border-radius: 8px;
    border: 1.5px solid #bdc3c7;
    font-size: 1rem;
    box-sizing: border-box;
    transition: border-color 0.3s ease;
  }
  input[type=text]:focus, select:focus, textarea:focus {
    border-color: #2980b9;
    outline: none;
  }
  button {
    background-color: #2980b9;
    border: none;
    color: white;
    font-weight: 600;
    border-radius: 10px;
    padding: 12px 25px;
    cursor: pointer;
    font-size: 1rem;
    transition: background-color 0.3s ease;
    margin-top: 10px;
  }
  button:hover {
    background-color: #1c5980;
  }
  table {
    width: 100%;
    border-collapse: collapse;
    margin-top: 15px;
    margin-bottom: 30px;
  }
  th, td {
    border: 1px solid #ddd;
    padding: 8px 12px;
    text-align: center;
  }
  th {
    background-color: #3498db;
    color: white;
  }
  .inline {
    display: flex;
    gap: 15px;
  }
  .inline > * {
    flex: 1 1 0;
  }
  pre {
    background-color: #ecf0f1;
    padding: 15px;
    border-radius: 8px;
    font-family: monospace;
    font-size: 0.95rem;
    max-height: 200px;
    overflow: auto;
  }

</style>
</head>
<body>
  <h1>Relational Algebra Visualizer</h1>

  <div class="container" aria-label="Create Relation">
    <h2>Create Relation</h2>
    <label for="relation-name">Relation Name</label>
    <input type="text" id="relation-name" placeholder="e.g. Students" aria-describedby="relNameDesc" />
    <small id="relNameDesc">Enter a unique name for your relation/table.</small>

    <label for="relation-schema">Schema (comma-separated attribute names)</label>
    <input type="text" id="relation-schema" placeholder="e.g. ID,Name,Age,Major" aria-describedby="schemaDesc" />
    <small id="schemaDesc">Example: ID, Name, Age</small>

    <label for="relation-data">Data (one tuple per line, comma-separated)</label>
    <textarea id="relation-data" rows="5" placeholder="e.g. 1,John,20,CS&#10;2,Alice,22,Math"></textarea>

    <button onclick="createRelation()">Add Relation</button>
  </div>

  <div class="container" aria-label="Relational Algebra Operations">
    <h2>Apply Relational Algebra Operation</h2>

    <label for="operation-type">Operation</label>
    <select id="operation-type" onchange="updateOperationFields()">
      <option value="select">SELECT (σ)</option>
      <option value="project">PROJECT (π)</option>
      <option value="join">JOIN (⋈)</option>
      <option value="union">UNION (∪)</option>
      <option value="difference">DIFFERENCE (−)</option>
    </select>

    <div id="op-fields"></div>

    <button onclick="applyOperation()">Apply Operation</button>
  </div>

  <div class="container" aria-label="Result">
    <h2>Result Relation</h2>
    <div id="result-area">No operation applied yet.</div>
  </div>

<script>
  // Store relations as objects: {name, schema:[], tuples: []}
  let relations = {};

  function createRelation() {
    const name = document.getElementById('relation-name').value.trim();
    if(!name) {
      alert('Please enter relation name.');
      return;
    }
    if(relations[name]) {
      alert('Relation with this name already exists. Use a different name.');
      return;
    }

    const schemaRaw = document.getElementById('relation-schema').value.trim();
    if(!schemaRaw) {
      alert('Please enter schema attributes.');
      return;
    }
    const schema = schemaRaw.split(',').map(s => s.trim());
    if(schema.length === 0) {
      alert('Schema must have at least one attribute.');
      return;
    }

    const dataRaw = document.getElementById('relation-data').value.trim();
    const tuples = [];
    if(dataRaw) {
      const lines = dataRaw.split('\n');
      for(let line of lines) {
        let values = line.split(',').map(s => s.trim());
        if(values.length !== schema.length) {
          alert(`Number of values in a tuple must match the schema length (${schema.length}). Error in line: "${line}"`);
          return;
        }
        tuples.push(values);
      }
    }

    relations[name] = {name, schema, tuples};
    alert(`Relation "${name}" added.`);
    clearCreateFields();
    updateOperationFields();
    displayRelations();
  }

  function clearCreateFields() {
    document.getElementById('relation-name').value = '';
    document.getElementById('relation-schema').value = '';
    document.getElementById('relation-data').value = '';
  }

  function displayRelations() {
    // Optional: display list of created relations with their schemas
    const container = document.getElementById('result-area');
    let html = '<h3>Existing Relations</h3>';
    if(Object.keys(relations).length === 0) {
      html += '<p>No relations created yet.</p>';
    } else {
      for(let r of Object.values(relations)) {
        html += `<strong>${r.name}</strong>: (${r.schema.join(', ')})<br>`;
      }
    }
    container.innerHTML = html;
  }

  // Update fields for the selected operation dynamically
  function updateOperationFields() {
    const op = document.getElementById('operation-type').value;
    const container = document.getElementById('op-fields');

    // Clear existing fields
    container.innerHTML = '';

    if(op === 'select') {
      container.innerHTML = `
        <label for="select-rel">Relation</label>
        <select id="select-rel">${relationOptions()}</select>
        <label for="select-cond">Condition (e.g. Age > 20)</label>
        <input type="text" id="select-cond" placeholder="e.g. Age = CS or Age > 20" />
      `;
    } else if (op === 'project') {
      container.innerHTML = `
        <label for="project-rel">Relation</label>
        <select id="project-rel">${relationOptions()}</select>
        <label for="project-atts">Attributes (comma-separated)</label>
        <input type="text" id="project-atts" placeholder="e.g. Name, Major" />
      `;
    } else if (op === 'join') {
      container.innerHTML = `
        <label for="join-rel1">Relation 1</label>
        <select id="join-rel1">${relationOptions()}</select>
        <label for="join-rel2">Relation 2</label>
        <select id="join-rel2">${relationOptions()}</select>
        <label for="join-cond">Join Condition (e.g. Relation1.ID = Relation2.ID)</label>
        <input type="text" id="join-cond" placeholder="e.g. Relation1.ID = Relation2.ID" />
      `;
    } else if (op === 'union' || op === 'difference') {
      container.innerHTML = `
        <label for="rel-a">Relation A</label>
        <select id="rel-a">${relationOptions()}</select>
        <label for="rel-b">Relation B</label>
        <select id="rel-b">${relationOptions()}</select>
      `;
    }
  }

  function relationOptions() {
    return Object.keys(relations).map(name => `<option value="${name}">${name}</option>`).join('');
  }

  // Parse condition string for select and join (very basic parser)
  function evalCondition(tuple, schema, cond) {
    // cond example: Age > 20 or Major = CS
    // This parser supports simple "Attribute operator value" with =, >, <, >=, <=
    if(!cond) return true;

    const ops = ['>=', '<=', '!=', '=', '>', '<'];
    let opFound = null;
    for(let op of ops) {
      if(cond.includes(op)) {
        opFound = op;
        break;
      }
    }
    if(!opFound) return true; // no valid operator found

    let [left, right] = cond.split(opFound).map(s => s.trim());
    let idx = schema.indexOf(left);
    if(idx === -1) return false;

    let val = tuple[idx];
    right = right.replace(/^['"]|['"]$/g, ''); // remove quotes if any

    switch(opFound) {
      case '=': return val === right;
      case '!=': return val !== right;
      case '>': return !isNaN(val) && !isNaN(right) && Number(val) > Number(right);
      case '<': return !isNaN(val) && !isNaN(right) && Number(val) < Number(right);
      case '>=': return !isNaN(val) && !isNaN(right) && Number(val) >= Number(right);
      case '<=': return !isNaN(val) && !isNaN(right) && Number(val) <= Number(right);
      default: return false;
    }
  }

  function applyOperation() {
    const op = document.getElementById('operation-type').value;
    let result = null;

    if(op === 'select') {
      const relName = document.getElementById('select-rel')?.value;
      const cond = document.getElementById('select-cond')?.value.trim();
      if(!relName || !relations[relName]) {
        alert('Please select a valid relation.');
        return;
      }
      const rel = relations[relName];
      const filteredTuples = rel.tuples.filter(t => evalCondition(t, rel.schema, cond));
      result = {
        name: `σ_${cond}(${relName})`,
        schema: rel.schema,
        tuples: filteredTuples,
      };

    } else if (op === 'project') {
      const relName = document.getElementById('project-rel')?.value;
      const attStr = document.getElementById('project-atts')?.value.trim();
      if(!relName || !relations[relName]) {
        alert('Select a valid relation for projection.');
        return;
      }
      if(!attStr) {
        alert('Enter attributes to project.');
        return;
      }
      const rel = relations[relName];
      const atts = attStr.split(',').map(a => a.trim());
      // Check that all attrs exist in rel.schema
      for(let a of atts) {
        if(!rel.schema.includes(a)) {
          alert(`Attribute "${a}" not found in relation "${relName}".`);
          return;
        }
      }
      const projSchema = atts;
      // Project tuples and remove duplicates
      const projTuples = [];
      const seen = new Set();
      for(let t of rel.tuples) {
        const projT = atts.map(a => t[rel.schema.indexOf(a)]);
        const key = projT.join('|');
        if(!seen.has(key)) {
          seen.add(key);
          projTuples.push(projT);
        }
      }
      result = {
        name: `π_${atts.join(',')}(${relName})`,
        schema: projSchema,
        tuples: projTuples,
      };

    } else if (op === 'join') {
      const relName1 = document.getElementById('join-rel1')?.value;
      const relName2 = document.getElementById('join-rel2')?.value;
      const cond = document.getElementById('join-cond')?.value.trim();
      if(!relName1 || !relName2 || !relations[relName1] || !relations[relName2]) {
        alert('Choose valid relations for join.');
        return;
      }
      if(!cond) {
        alert('Enter join condition.');
        return;
      }
      const rel1 = relations[relName1];
      const rel2 = relations[relName2];

      // Parse condition like Relation1.ID = Relation2.ID
      let joinCols = cond.split('=').map(s => s.trim());
      if(joinCols.length !== 2) {
        alert('Invalid join condition format. Use Relation1.Attr = Relation2.Attr');
        return;
      }
      let [left, right] = joinCols;
      if(!left.includes('.') || !right.includes('.')) {
        alert('Join condition attributes must be prefixed with relation name e.g. Relation1.ID = Relation2.ID');
        return;
      }
      let [lRelName, lAttr] = left.split('.');
      let [rRelName, rAttr] = right.split('.');
      if(lRelName !== relName1 || rRelName !== relName2) {
        alert('Join condition relations do not match selected relations.');
        return;
      }
      const lIdx = rel1.schema.indexOf(lAttr);
      const rIdx = rel2.schema.indexOf(rAttr);
      if(lIdx === -1 || rIdx === -1) {
        alert('Join attributes not found in respective relations.');
        return;
      }

      // Build joined schema (excluding duplicate join attr from second relation)
      const joinedSchema = [...rel1.schema, ...rel2.schema.filter((a, i) => i !== rIdx)];
      const joinedTuples = [];

      // Nested loop join for demo (could optimize)
      for(let t1 of rel1.tuples) {
        for(let t2 of rel2.tuples) {
          if(t1[lIdx] === t2[rIdx]) {
            const tuple = [...t1, ...t2.filter((_, i) => i !== rIdx)];
            joinedTuples.push(tuple);
          }
        }
      }

      result = {
        name: `(${relName1} ⋈ ${relName2})`,
        schema: joinedSchema,
        tuples: joinedTuples,
      };

    } else if (op === 'union' || op === 'difference') {
      const relNameA = document.getElementById('rel-a')?.value;
      const relNameB = document.getElementById('rel-b')?.value;
      if(!relNameA || !relNameB || !relations[relNameA] || !relations[relNameB]) {
        alert('Select valid relations.');
        return;
      }
      const relA = relations[relNameA];
      const relB = relations[relNameB];
      // schemas must be identical (attribute names and count)
      if(relA.schema.length !== relB.schema.length ||
        !relA.schema.every((a,i) => a === relB.schema[i])){
        alert('Relations must have identical schemas for Union or Difference.');
        return;
      }
      let tuplesResult = [];
      const setA = new Set(relA.tuples.map(t => t.join('|')));
      const setB = new Set(relB.tuples.map(t => t.join('|')));

      if(op === 'union') {
        const unionSet = new Set([...setA, ...setB]);
        tuplesResult = Array.from(unionSet).map(str => str.split('|'));
      } else if (op === 'difference') {
        const diffSet = new Set([...setA]);
        for(let val of setB) {
          diffSet.delete(val);
        }
        tuplesResult = Array.from(diffSet).map(str => str.split('|'));
      }

      result = {
        name: `${op === 'union' ? '∪' : '−'}(${relNameA},${relNameB})`,
        schema: relA.schema,
        tuples: tuplesResult,
      };
    }

    if(result) {
      displayRelation(result);
    }
  }

  function displayRelation(rel) {
    const container = document.getElementById('result-area');
    let html = `<h3>${rel.name}</h3>`;
    if(rel.tuples.length === 0) {
      html += '<p><em>No tuples.</em></p>';
    } else {
      html += '<table><thead><tr>';
      for(let attr of rel.schema) {
        html += `<th>${attr}</th>`;
      }
      html += '</tr></thead><tbody>';
      rel.tuples.forEach(tuple => {
        html += '<tr>';
        tuple.forEach(val => html += `<td>${val}</td>`);
        html += '</tr>';
      });
      html += '</tbody></table>';
    }
    container.innerHTML = html;
  }

  // Initialize operation fields at page load
  updateOperationFields();
  displayRelations();
</script>

</body>
</html>
