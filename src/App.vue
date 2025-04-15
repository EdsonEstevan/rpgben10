<template>
  <div id="app" class="container">
    <h1>Ben10 RPG - Criador de Fichas</h1>
    
    <!-- Seção de Criação Básica -->
    <div class="creation-section">
      <h2>Informações Básicas</h2>
      <div class="form-group">
        <label>Nome:</label>
        <input v-model="character.name" type="text">
      </div>
      <div class="form-group">
        <label>Idade:</label>
        <input v-model="character.age" type="number">
      </div>
      <div class="form-group">
        <label>Altura:</label>
        <input v-model="character.height" type="text">
      </div>
      <div class="form-group">
        <label>Peso:</label>
        <input v-model="character.weight" type="text">
      </div>
      <div class="form-group">
  <label>Espécie:</label>
  <select v-model="character.species" @change="applySpeciesBonuses">
    <option value="">Selecione uma espécie</option>
    <option 
      v-for="(bonuses, speciesName) in alienSpecies" 
      :key="speciesName" 
      :value="speciesName"
    >
      {{ speciesName }}
    </option>
  </select>
  
  <!-- Adicione este botão -->
  <div class="hybrid-toggle">
    <label>
      <input type="checkbox" v-model="character.isHybrid"> Híbrido (Humano + espécie selecionada)
    </label>
  </div>
  
  <div v-if="character.species && alienSpecies[character.species]" class="species-bonuses">
    <h4>Bônus da Espécie:</h4>
    <p v-for="(value, attr) in alienSpecies[character.species]" :key="attr">
      {{ attr.toUpperCase() }}: +{{ value }}
    </p>
  </div>
</div>
      <div class="form-group">
        <label>Academia:</label>
        <select v-model="character.academy" @change="updateAcademySpecifics">
          <option value="Adestrador">Adestrador</option>
          <option value="Encanador">Encanador</option>
          <option value="Mágico">Mágico</option>
          <option value="Relojoeiro">Relojoeiro</option>
        </select>
      </div>
      <div class="form-group">
        <label>Hierarquia:</label>
        <select v-model="character.hierarchy" @change="updateAttributePoints">
          <option value="Cadete">Cadete</option>
          <option value="Cadete Oficializado">Cadete Oficializado</option>
          <option value="Oficial">Oficial</option>
        </select>
      </div>
      <div class="form-group">
        <label>Talento:</label>
        <select v-model="character.talent">
          <option value="">Nenhum (+8 MOD atributo)</option>
          <option value="Ápice Físico">Ápice Físico</option>
          <option value="Aptidão Mágica">Aptidão Mágica</option>
          <option value="Gênio">Gênio</option>
          <option value="Memória Eidética">Memória Eidética</option>
          <option value="Mutante">Mutante</option>
          <option value="Sociável">Sociável</option>
        </select>
      </div>
    </div>
  
      <!-- Atributos -->
      <div class="attributes-section">
        <h2>Atributos (Pontos disponíveis: {{ attributePoints }})</h2>
        <div class="attributes-grid">
          <div class="attribute" v-for="(value, attr) in character.attributes" :key="attr">
            <label>{{ getAttributeName(attr) }}:</label>
            <input 
              type="number" 
              v-model.number="character.attributes[attr]" 
              :min="getMinAttribute(attr)" 
              :max="getMaxAttribute(attr)"
              @change="updateAttributePoints"
            >
            <span class="modifier">Mod: {{ calculateModifier(value) }}</span>
          </div>
        </div>
      </div>
  
      <!-- Status Calculados -->
      <div class="stats-section">
        <h2>Status</h2>
        <div class="stats-grid">
          <div class="stat">
            <label>PV:</label>
            <span>{{ calculatePV() }}</span>
          </div>
          <div class="stat">
            <label>PE:</label>
            <span>{{ calculatePE() }}</span>
          </div>
          <div class="stat">
            <label>PA:</label>
            <span>{{ calculatePA() }}</span>
          </div>
          <div class="stat">
            <label>PP:</label>
            <span>{{ calculatePP() }}</span>
          </div>
          <div class="stat">
            <label>PC:</label>
            <span>{{ calculatePC() }}</span>
          </div>
        </div>
        <div class="stats-grid">
          <div class="stat">
            <label>CR Organismo:</label>
            <span>{{ calculateCROrganismo() }}</span>
          </div>
          <div class="stat">
            <label>CR Corpo:</label>
            <span>{{ calculateCRCorpo() }}</span>
          </div>
          <div class="stat">
            <label>CR Mente:</label>
            <span>{{ calculateCRMente() }}</span>
          </div>
        </div>
      </div>
  
      <!-- Seções Específicas por Academia -->
      <div v-if="character.academy === 'Adestrador'" class="academy-section">
        <h2>Adestrador</h2>
        <div class="form-group">
          <label>Pontos Omni (PO):</label>
          <input v-model="character.adestrador.po" type="number" min="0">
        </div>
        
        <h3>Nemetrix</h3>
        <div class="form-group">
          <label>Funções:</label>
          <textarea v-model="character.adestrador.funcoes"></textarea>
        </div>
        
        <h3>Lista de Aliens</h3>
        <div v-for="(alien, index) in character.adestrador.aliens" :key="index" class="alien-entry">
          <input v-model="alien.name" type="text" placeholder="Nome do Alien">
          <input v-model="alien.energy" type="number" placeholder="Energia" min="0">
          <button @click="removeAlien(index)">Remover</button>
        </div>
        <button @click="addAlien">Adicionar Alien</button>
      </div>
  
      <div v-else-if="character.academy === 'Encanador'" class="academy-section">
        <h2>Encanador</h2>
        <div class="form-group">
          <label>Preparo:</label>
          <span>{{ calculatePreparo() }}</span>
        </div>
      </div>
  
      <div v-else-if="character.academy === 'Mágico'" class="academy-section">
        <h2>Mágico</h2>
        <div class="form-group">
          <label>Nível Mágico (NM):</label>
          <select v-model="character.magico.nm">
            <option value="0">Nível 0</option>
            <option value="1">Nível 1</option>
            <option value="2">Nível 2</option>
            <option value="3">Nível 3</option>
            <option value="Mestre">Mestre</option>
          </select>
        </div>
        <div class="form-group">
          <label>Pontos de Mana (PM):</label>
          <span>{{ calculatePM() }}</span>
        </div>
        <div class="form-group">
          <label>Magias Memorizadas (MM):</label>
          <span>{{ calculateMM() }}</span>
        </div>
        
        <h3>Lista de Magias Conhecidas</h3>
        <div v-for="(spell, index) in character.magico.magias" :key="index" class="spell-entry">
          <input v-model="spell.name" type="text" placeholder="Nome da Magia">
          <select v-model="spell.level">
            <option value="0">Nível 0</option>
            <option value="1">Nível 1</option>
            <option value="2">Nível 2</option>
            <option value="3">Nível 3</option>
            <option value="Mestre">Mestre</option>
          </select>
          <button @click="removeSpell(index)">Remover</button>
        </div>
        <button @click="addSpell">Adicionar Magia</button>
      </div>
  
      <div v-else-if="character.academy === 'Relojoeiro'" class="academy-section">
        <h2>Relojoeiro</h2>
        <div class="form-group">
          <label>Pontos Omni (PO):</label>
          <input v-model="character.relojoeiro.po" type="number" min="0">
        </div>
        <div class="form-group">
          <label>Energia Omni (EO):</label>
          <span>100 / 100</span>
        </div>
        
        <h3>Decatrix</h3>
        <div class="form-group">
          <label>Funções:</label>
          <textarea v-model="character.relojoeiro.funcoes"></textarea>
        </div>
        
        <h3>Lista de Aliens</h3>
        <div v-for="(alien, index) in character.relojoeiro.aliens" :key="index" class="alien-entry">
          <input v-model="alien.name" type="text" placeholder="Nome do Alien">
          <input v-model="alien.energy" type="number" placeholder="Energia" min="0">
          <button @click="removeDecatrixAlien(index)">Remover</button>
        </div>
        <button @click="addDecatrixAlien">Adicionar Alien</button>
      </div>
  
      <!-- Perícias -->
      <div class="skills-section">
        <h2>Perícias</h2>
        <div class="skills-category">
          <h3>Básicas</h3>
          <div v-for="(skill, index) in character.skills.basicas" :key="'basica-'+index" class="skill-entry">
            <input v-model="skill.name" type="text" placeholder="Nome da Perícia">
            <select v-model="skill.level">
              <option value="0">Nível 0</option>
              <option value="1">Nível 1</option>
              <option value="2">Nível 2</option>
              <option value="3">Nível 3</option>
              <option value="4">Nível 4</option>
            </select>
            <button @click="removeSkill('basicas', index)">Remover</button>
          </div>
          <button @click="addSkill('basicas')">Adicionar Perícia Básica</button>
        </div>
        
        <div class="skills-category">
          <h3>Avançadas</h3>
          <div v-for="(skill, index) in character.skills.avancadas" :key="'avancada-'+index" class="skill-entry">
            <input v-model="skill.name" type="text" placeholder="Nome da Perícia">
            <select v-model="skill.level">
              <option value="0">Nível 0</option>
              <option value="1">Nível 1</option>
              <option value="2">Nível 2</option>
              <option value="3">Nível 3</option>
              <option value="4">Nível 4</option>
            </select>
            <button @click="removeSkill('avancadas', index)">Remover</button>
          </div>
          <button @click="addSkill('avancadas')">Adicionar Perícia Avançada</button>
        </div>
        
        <div class="skills-category">
          <h3>Circunstanciais</h3>
          <div v-for="(skill, index) in character.skills.circunstanciais" :key="'circunstancial-'+index" class="skill-entry">
            <input v-model="skill.name" type="text" placeholder="Nome da Perícia">
            <button @click="removeSkill('circunstanciais', index)">Remover</button>
          </div>
          <button @click="addSkill('circunstanciais')">Adicionar Perícia Circunstancial</button>
        </div>
      </div>
  
      <!-- Técnicas -->
      <div class="techniques-section">
        <h2>Técnicas</h2>
        <div v-for="(technique, index) in character.techniques" :key="index" class="technique-entry">
          <input v-model="technique.name" type="text" placeholder="Nome da Técnica">
          <input v-model="technique.complexity" type="number" placeholder="Complexidade" min="1" max="20">
          <button @click="removeTechnique(index)">Remover</button>
        </div>
        <button @click="addTechnique">Adicionar Técnica</button>
      </div>
  
      <!-- Anotações -->
      <div class="notes-section">
        <h2>Anotações</h2>
        <div class="form-group">
          <label>Itens:</label>
          <textarea v-model="character.notes.itens"></textarea>
        </div>
        <div class="form-group">
          <label>Contatos (NPCs e PJs):</label>
          <textarea v-model="character.notes.contatos"></textarea>
        </div>
        <div class="form-group">
          <label>Anotações Relevantes:</label>
          <textarea v-model="character.notes.anotacoes"></textarea>
        </div>
        <div class="form-group">
          <label>História:</label>
          <textarea v-model="character.notes.historia"></textarea>
        </div>
      </div>
  
      <!-- Botões de Ação -->
      <div class="action-buttons">
        <button @click="saveCharacter">Salvar Ficha</button>
        <button @click="resetCharacter">Nova Ficha</button>
        <button @click="printCharacter">Imprimir Ficha</button>
      </div>
    </div>
  </template>
  
  <script>
import { alienSpecies } from './data/alienData';


export default {
  name: 'App',
  data() {
    return {
      alienSpecies: alienSpecies, // Usamos diretamente o objeto importado
      character: {
      isHybrid: false, // Adicione esta linha
      species: '',
        attributes: {
          for: 0,
          des: 0,
          con: 0,
          res: 0,
          int: 0,
          sab: 0,
          car: 0,
          mal: 0
        },
          powers: [],
          weaknesses: '',
          skills: {
            basicas: [],
            avancadas: [],
            circunstanciais: []
          },
          techniques: [],
          notes: {
            itens: '',
            contatos: '',
            anotacoes: '',
            historia: ''
          },
          adestrador: {
            po: 50,
            funcoes: 'Funções Básicas [...]',
            aliens: []
          },
          encanador: {
            preparo: 0
          },
          magico: {
            nm: '0',
            pm: 0,
            mm: 0,
            magias: []
          },
          relojoeiro: {
            po: 80,
            eo: 100,
            funcoes: 'Funções Básicas [...]',
            aliens: []
          }
        },
        attributePoints: 24,
        speciesList: [
          'Aerofibiano', 'Anodita', 'Antigravitesla', 'Arachnachimp', 'Argit', 'Amperis', 'Apoplixianos', 
          'Atômico', 'Atrocianos', 'Basalt', 'Bigotoides', 'Biosovortianos', 'Bob The Blob', 'Calbreus', 
          'Cascanos', 'Cerebrocrustaceanos', 'Cephalod-AE', 'Chimeras Sui Generis', 'Citrakayahs', 
          'Cocorocoide', 'Conductoides', 'Chronosapianos', 'Churls', 'Crystalsapiens', 'Dragão Alien', 
          'Detrovites', 'Ectonuritas', 'Ekoplektoid', 'Escarabola', 'Espantoide', 'Floraunas', 'Fulminis', 
          'Galileanos', 'Galvanianos', 'Gimlinopithecus', 'Geochelone Aerios', 'Gourmandos', 'Humanos', 
          'Humanos de Ledgerdomain', 'Inkursianos', 'Inkthiperambuloides', 'Kinecelerandos', 'Kraahos', 
          'Lenopanos', 'Lepidoptereanos', 'Lewodanianos', 'Loboans', 'Mecamorfos Galvanics', 
          'Merlini Sapiens', 'Metanosianos', 'Muroids', 'Nanochip Humano', 'Necrofriggianos', 'Nemoinas', 
          'Nosedeenianos', 'Opiticoides', 'Orishanos', 'Orthopterranos', 'Oryctinis', 'Pantophages', 
          'Pelarotas Arburianos', 'Petrosapiens', 'Pisccis Premann', 'Pisccis Volanns', 'Pitorovosaurianos', 
          'Planchaküles', 'Podrão', 'Polar Manzardills', 'Polymorfos', 'Protosts', 'Prypiatosians B', 
          'Pugnivoros', 'Pyronitas', 'Revonnah Ganders', 'Sapiens Celestiais', 'Segmento Sapiens', 
          'Snidellerodontu', 'Soberanos', 'Solinoides', 'Sonorosianos', 'Spheroides', 'Splixions', 
          'Sylonnoids', 'Talpaedanos', 'Tetramandos', 'Thalassiano', 'Thep Khufans', 'To,Kustares', 
          'Transylianos', 'Últimos', 'Uxorites', 'Vaxasaurianos', 'Vladats', 'Vreedls', 'Vulpimanceres', 
          'Xerges', 'Zaroffianos'
        ]
      };
    },
    watch: {
  'character.isHybrid': {
    handler() {
      if (this.character.species) {
        this.applySpeciesBonuses();
      }
    },
    immediate: true
  },
  'character.species'() {
    this.applySpeciesBonuses();
  }
},
methods: {
  applySpeciesBonuses() {
    if (!this.character.species || !this.alienSpecies[this.character.species]) return;

    const speciesBonuses = this.alienSpecies[this.character.species];
    const humanBonuses = this.alienSpecies['Humano'] || {};
    
    // Calcula os bônus finais
    const finalBonuses = {};
    for (const attr in speciesBonuses) {
      if (this.character.isHybrid && this.character.species !== 'Humano') {
        // Média entre espécie selecionada e humano
        finalBonuses[attr] = Math.round(
          (speciesBonuses[attr] + (humanBonuses[attr] || 0)) / 2
        );
      } else {
        // Bônus normal da espécie
        finalBonuses[attr] = speciesBonuses[attr];
      }
    }

    // Aplica os bônus
    for (const attr in finalBonuses) {
      if (attr in this.character.attributes) {
        this.character.attributes[attr] = finalBonuses[attr];
      }
    }

    this.updateAttributePoints();
  },

      getAttributeName(attr) {
        const names = {
          for: 'FOR (Força)',
          des: 'DES (Destreza)',
          con: 'CON (Constituição)',
          res: 'RES (Resistência)',
          int: 'INT (Inteligência)',
          sab: 'SAB (Sabedoria)',
          car: 'CAR (Carisma)',
          mal: 'MAL (Malícia)'
        };
        return names[attr] || attr;
      },
      calculateModifier(value) {
        return Math.floor((value - 0) / 1);
      },
      updateAttributePoints() {
  let totalUsed = 0;
  const baseAttributes = this.getBaseAttributes(); // Adicionaremos este método
  
  for (const attr in baseAttributes) {
    totalUsed += baseAttributes[attr];
  }
  
  let basePoints = 24;
  if (this.character.hierarchy === 'Cadete Oficializado') basePoints = 34;
  if (this.character.hierarchy === 'Oficial') basePoints = 44;
  
  this.attributePoints = basePoints - totalUsed;
},

getBaseAttributes() {
  const baseAttributes = {};
  const bonuses = this.character.species ? this.alienSpecies[this.character.species] || {} : {};
  
  for (const attr in this.character.attributes) {
    const bonus = bonuses[attr] || 0;
    baseAttributes[attr] = Math.max(0, this.character.attributes[attr] - bonus);
  }
  
  return baseAttributes;
},
      getMinAttribute() {
      return -4;
    },
    getMaxAttribute(attr) {
  const baseMax = 10; // Máximo para pontos distribuídos
  const bonus = this.character.species && this.alienSpecies[this.character.species] 
    ? this.alienSpecies[this.character.species][attr] || 0 
    : 0;
  
  return baseMax + bonus; // Permite que o total ultrapasse 10 com bônus
},
      calculatePV() {
        return (this.character.attributes.con * 5) + 25;
      },
      calculatePE() {
        return Math.floor((this.character.attributes.for + this.character.attributes.des + 
                         this.character.attributes.con + this.character.attributes.res) / 4);
      },
      calculatePA() {
        return Math.floor((this.character.attributes.des + this.character.attributes.sab) / 20) + 1;
      },
      calculatePP() {
        return 10 + this.character.attributes.sab;
      },
      calculatePC() {
        return 10 + this.character.attributes.car;
      },
      calculateCROrganismo() {
        return this.character.attributes.con + 15;
      },
      calculateCRCorpo() {
        return Math.floor((this.character.attributes.for + this.character.attributes.res) / 2) + 10;
      },
      calculateCRMente() {
        return Math.floor((this.character.attributes.sab + this.character.attributes.int) / 2) + 10;
      },
      calculatePreparo() {
        return Math.floor((this.character.attributes.int + this.character.attributes.sab) / 2);
      },
      calculatePM() {
        return (this.character.attributes.con + this.character.attributes.int) * 2;
      },
      calculateMM() {
        return Math.floor((this.character.attributes.int + this.character.attributes.sab) / 2);
      },
      updateAcademySpecifics() {
        // Reset academy-specific data when academy changes
        this.character.adestrador = {
          po: 50,
          funcoes: 'Funções Básicas [...]',
          aliens: []
        };
        this.character.encanador = {
          preparo: this.calculatePreparo()
        };
        this.character.magico = {
          nm: '0',
          pm: this.calculatePM(),
          mm: this.calculateMM(),
          magias: []
        };
        this.character.relojoeiro = {
          po: 80,
          eo: 100,
          funcoes: 'Funções Básicas [...]',
          aliens: []
        };
      },
      addAlien() {
        this.character.adestrador.aliens.push({
          name: '',
          energy: 0
        });
      },
      removeAlien(index) {
        this.character.adestrador.aliens.splice(index, 1);
      },
      addDecatrixAlien() {
        this.character.relojoeiro.aliens.push({
          name: '',
          energy: 0
        });
      },
      removeDecatrixAlien(index) {
        this.character.relojoeiro.aliens.splice(index, 1);
      },
      addSpell() {
        this.character.magico.magias.push({
          name: '',
          level: '0'
        });
      },
      removeSpell(index) {
        this.character.magico.magias.splice(index, 1);
      },
      addSkill(type) {
        this.character.skills[type].push({
          name: '',
          level: '0'
        });
      },
      removeSkill(type, index) {
        this.character.skills[type].splice(index, 1);
      },
      addTechnique() {
        this.character.techniques.push({
          name: '',
          complexity: 1
        });
      },
      removeTechnique(index) {
        this.character.techniques.splice(index, 1);
      },
      saveCharacter() {
        const characterData = JSON.stringify(this.character);
        localStorage.setItem('ben10rpg_character', characterData);
        alert('Ficha salva com sucesso!');
      },
      resetCharacter() {
        if (confirm('Tem certeza que deseja criar uma nova ficha? Todos os dados não salvos serão perdidos.')) {
          this.character = {
            name: '',
            age: '',
            height: '',
            weight: '',
            species: '',
            academy: '',
            hierarchy: 'Cadete',
            talent: '',
            attributes: {
              for: 0,
              des: 0,
              con: 0,
              res: 0,
              int: 0,
              sab: 0,
              car: 0,
              mal: 0
            },
            powers: [],
            weaknesses: '',
            skills: {
              basicas: [],
              avancadas: [],
              circunstanciais: []
            },
            techniques: [],
            notes: {
              itens: '',
              contatos: '',
              anotacoes: '',
              historia: ''
            },
            adestrador: {
              po: 50,
              funcoes: 'Funções Básicas [...]',
              aliens: []
            },
            encanador: {
              preparo: 0
            },
            magico: {
              nm: '0',
              pm: 0,
              mm: 0,
              magias: []
            },
            relojoeiro: {
              po: 80,
              eo: 100,
              funcoes: 'Funções Básicas [...]',
              aliens: []
            }
          };
          this.attributePoints = 24;
        }
      },
      printCharacter() {
        window.print();
      }
    },
    mounted() {
      const savedCharacter = localStorage.getItem('ben10rpg_character');
      if (savedCharacter) {
        this.character = JSON.parse(savedCharacter);
        this.updateAttributePoints();
      }
    }
  };
  </script>
  
  <style>
  .species-bonuses {
  margin-top: 10px;
  padding: 10px;
  background-color: #f0f0f0;
  border-radius: 5px;
}
.species-bonuses h4 {
  margin-top: 0;
}
.species-bonuses p {
  margin: 5px 0;
}
  /* Estilos básicos */
  body {
    font-family: Arial, sans-serif;
    line-height: 1.6;
    color: #333;
    max-width: 1200px;
    margin: 0 auto;
    padding: 20px;
    background-color: #f5f5f5;
  }
  
  .container {
    background-color: white;
    padding: 20px;
    border-radius: 8px;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  }
  
  h1, h2, h3 {
    color: #2c3e50;
    border-bottom: 1px solid #eee;
    padding-bottom: 10px;
  }
  
  .form-group {
    margin-bottom: 15px;
  }
  
  label {
    display: block;
    margin-bottom: 5px;
    font-weight: bold;
  }
  
  input[type="text"],
  input[type="number"],
  select,
  textarea {
    width: 100%;
    padding: 8px;
    border: 1px solid #ddd;
    border-radius: 4px;
    box-sizing: border-box;
  }
  
  button {
    background-color: #3498db;
    color: white;
    border: none;
    padding: 8px 15px;
    border-radius: 4px;
    cursor: pointer;
    margin-right: 10px;
    margin-bottom: 10px;
  }
  
  button:hover {
    background-color: #2980b9;
  }
  
  /* Layout de seções */
  .creation-section,
  .attributes-section,
  .stats-section,
  .academy-section,
  .skills-section,
  .techniques-section,
  .notes-section {
    margin-bottom: 30px;
    padding: 15px;
    background-color: #f9f9f9;
    border-radius: 5px;
  }
  
  /* Grid de atributos */
  .attributes-grid {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 15px;
  }
  
  .attribute {
    display: flex;
    align-items: center;
    gap: 10px;
  }
  
  .attribute input {
    width: 60px;
  }
  
  .modifier {
    font-style: italic;
    color: #666;
  }
  
  /* Grid de status */
  .stats-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
    gap: 15px;
  }
  
  .stat {
    display: flex;
    justify-content: space-between;
  }
  
  /* Entradas de habilidades e técnicas */
  .skill-entry,
  .technique-entry,
  .alien-entry,
  .spell-entry {
    display: flex;
    gap: 10px;
    margin-bottom: 10px;
    align-items: center;
  }
  
  .skill-entry input,
  .technique-entry input,
  .alien-entry input,
  .spell-entry input {
    flex-grow: 1;
  }
  
  /* Botões de ação */
  .action-buttons {
    margin-top: 30px;
    text-align: center;
  }
  
  /* Responsividade */
  @media (max-width: 768px) {
    .attributes-grid {
      grid-template-columns: 1fr;
    }
    
    .stats-grid {
      grid-template-columns: 1fr;
    }
    
    .skill-entry,
    .technique-entry,
    .alien-entry,
    .spell-entry {
      flex-direction: column;
      align-items: flex-start;
    }
  }
  
  @media print {
    button {
      display: none;
    }
    
    .container {
      box-shadow: none;
      padding: 0;
    }
  }
  </style>