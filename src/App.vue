<template>
  <div id="app">
    <div class="container">
      <h1>Ben10 RPG - Criador de Fichas</h1>
      
      <!-- Seleção Básica do Personagem -->
      <div class="section">
        <h2>Informações Básicas</h2>
        <div class="form-group">
          <label>Nome:</label>
          <input v-model="character.name" type="text">
        </div>
        <div class="form-group">
          <label>Idade:</label>
          <input v-model="character.age" type="number" min="0">
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
          <select v-model="character.species">
            
          </select>
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
      <div class="section">
        <h2>Atributos</h2>
        <p>Pontos disponíveis: {{ attributePoints }}</p>
        <div class="attributes-grid">
          <div class="attribute" v-for="(value, name) in character.attributes" :key="name">
            <label>{{ name }}:</label>
            <input type="number" v-model.number="character.attributes[name]" min="-4" :max="10 + getSpeciesBonus(name)">
            <span>Bônus racial: {{ getSpeciesBonus(name) }}</span>
          </div>
        </div>
      </div>

      <!-- Poderes -->
      <div class="section">
        <h2>Poderes</h2>
        <div class="form-group">
          <label>Nome do Poder:</label>
          <input v-model="newPower.name" type="text">
        </div>
        <div class="form-group">
          <label>Valor:</label>
          <input v-model.number="newPower.value" type="number" min="0" max="80">
        </div>
        <div class="form-group">
          <label>Habilidade:</label>
          <textarea v-model="newPower.ability"></textarea>
        </div>
        <div class="form-group">
          <label>Limitações:</label>
          <textarea v-model="newPower.limitations"></textarea>
        </div>
        <button @click="addPower">Adicionar Poder</button>
        
        <div v-for="(power, index) in character.powers" :key="index" class="power">
          <h3>{{ power.name }} [{{ power.value }}]</h3>
          <p><strong>Habilidade:</strong> {{ power.ability }}</p>
          <p><strong>Limitações:</strong> {{ power.limitations }}</p>
          <button @click="removePower(index)">Remover</button>
        </div>
      </div>

      <!-- Fraquezas -->
      <div class="section">
        <h2>Fraquezas</h2>
        <div class="form-group">
          <textarea v-model="character.weaknesses"></textarea>
        </div>
      </div>

      <!-- Perícias -->
      <div class="section">
        <h2>Perícias</h2>
        <div class="skills-section">
          <div class="skill-type">
            <h3>Básicas</h3>
            <div v-for="(skill, index) in character.skills.basic" :key="'basic-'+index" class="skill">
              <label>{{ skill.name }}:</label>
              <select v-model="skill.level">
                <option value="0">N0</option>
                <option value="1">N1</option>
                <option value="2">N2</option>
                <option value="3">N3</option>
                <option value="4">N4</option>
              </select>
              <button @click="removeSkill('basic', index)">Remover</button>
            </div>
            <div class="add-skill">
              <input v-model="newBasicSkill" type="text" placeholder="Nova perícia básica">
              <button @click="addSkill('basic')">Adicionar</button>
            </div>
          </div>

          <div class="skill-type">
            <h3>Avançadas</h3>
            <div v-for="(skill, index) in character.skills.advanced" :key="'advanced-'+index" class="skill">
              <label>{{ skill.name }}:</label>
              <select v-model="skill.level">
                <option value="0">N0</option>
                <option value="1">N1</option>
                <option value="2">N2</option>
                <option value="3">N3</option>
                <option value="4">N4</option>
              </select>
              <button @click="removeSkill('advanced', index)">Remover</button>
            </div>
            <div class="add-skill">
              <input v-model="newAdvancedSkill" type="text" placeholder="Nova perícia avançada">
              <button @click="addSkill('advanced')">Adicionar</button>
            </div>
          </div>

          <div class="skill-type">
            <h3>Circunstanciais</h3>
            <div v-for="(skill, index) in character.skills.circumstantial" :key="'circumstantial-'+index" class="skill">
              <label>{{ skill.name }}:</label>
              <select v-model="skill.level">
                <option value="0">N0</option>
                <option value="1">N1</option>
              </select>
              <button @click="removeSkill('circumstantial', index)">Remover</button>
            </div>
            <div class="add-skill">
              <input v-model="newCircumstantialSkill" type="text" placeholder="Nova perícia circunstancial">
              <button @click="addSkill('circumstantial')">Adicionar</button>
            </div>
          </div>
        </div>
      </div>

      <!-- Técnicas -->
      <div class="section">
        <h2>Técnicas</h2>
        <div class="form-group">
          <input v-model="newTechnique" type="text" placeholder="Nova técnica">
          <button @click="addTechnique">Adicionar</button>
        </div>
        <div v-for="(technique, index) in character.techniques" :key="index" class="technique">
          <p>{{ technique }}</p>
          <button @click="removeTechnique(index)">Remover</button>
        </div>
      </div>

      <!-- Seções específicas da academia -->
      <div v-if="character.academy === 'Adestrador'" class="section academy-specific">
        <h2>Adestrador</h2>
        <div class="form-group">
          <label>Pontos Omni (PO):</label>
          <input v-model.number="character.academyData.po" type="number" min="0">
        </div>
        
        <h3>Perícias de Adestrador</h3>
        <div class="skills-section">
          <div class="skill-type">
            <h4>Básicas</h4>
            <div v-for="(skill, index) in character.academyData.skills.basic" :key="'trainer-basic-'+index" class="skill">
              <label>{{ skill.name }}:</label>
              <select v-model="skill.level">
                <option value="0">N0</option>
                <option value="1">N1</option>
                <option value="2">N2</option>
                <option value="3">N3</option>
                <option value="4">N4</option>
              </select>
              <button @click="removeAcademySkill('basic', index)">Remover</button>
            </div>
            <div class="add-skill">
              <input v-model="newTrainerBasicSkill" type="text" placeholder="Nova perícia básica">
              <button @click="addAcademySkill('basic')">Adicionar</button>
            </div>
          </div>

          <div class="skill-type">
            <h4>Avançadas</h4>
            <div v-for="(skill, index) in character.academyData.skills.advanced" :key="'trainer-advanced-'+index" class="skill">
              <label>{{ skill.name }}:</label>
              <select v-model="skill.level">
                <option value="0">N0</option>
                <option value="1">N1</option>
                <option value="2">N2</option>
                <option value="3">N3</option>
                <option value="4">N4</option>
              </select>
              <button @click="removeAcademySkill('advanced', index)">Remover</button>
            </div>
            <div class="add-skill">
              <input v-model="newTrainerAdvancedSkill" type="text" placeholder="Nova perícia avançada">
              <button @click="addAcademySkill('advanced')">Adicionar</button>
            </div>
          </div>
        </div>

        <h3>Técnicas de Adestrador</h3>
        <div class="form-group">
          <input v-model="newTrainerTechnique" type="text" placeholder="Nova técnica">
          <button @click="addAcademyTechnique">Adicionar</button>
        </div>
        <div v-for="(technique, index) in character.academyData.techniques" :key="'trainer-tech-'+index" class="technique">
          <p>{{ technique }}</p>
          <button @click="removeAcademyTechnique(index)">Remover</button>
        </div>

        <h3>Nemetrix</h3>
        <div class="form-group">
          <label>Funções:</label>
          <textarea v-model="character.academyData.nemetrix.functions"></textarea>
        </div>
        
        <h4>Lista de Aliens</h4>
        <div v-for="(alien, index) in character.academyData.nemetrix.aliens" :key="'alien-'+index" class="alien">
          <div class="form-group">
            <label>Nome:</label>
            <input v-model="alien.name" type="text">
          </div>
          <div class="form-group">
            <label>Espécie:</label>
            <input v-model="alien.species" type="text">
          </div>
          <button @click="removeAlien(index)">Remover</button>
        </div>
        <button @click="addAlien">Adicionar Alien</button>
      </div>

      <div v-if="character.academy === 'Encanador'" class="section academy-specific">
        <h2>Encanador</h2>
        <div class="form-group">
          <label>Preparo:</label>
          <input v-model.number="character.academyData.preparation" type="number" min="0">
        </div>
        
        <h3>Perícias de Encanador</h3>
        <div class="skills-section">
          <div class="skill-type">
            <h4>Básicas</h4>
            <div v-for="(skill, index) in character.academyData.skills.basic" :key="'plumber-basic-'+index" class="skill">
              <label>{{ skill.name }}:</label>
              <select v-model="skill.level">
                <option value="0">N0</option>
                <option value="1">N1</option>
                <option value="2">N2</option>
                <option value="3">N3</option>
                <option value="4">N4</option>
              </select>
              <button @click="removeAcademySkill('basic', index)">Remover</button>
            </div>
            <div class="add-skill">
              <input v-model="newPlumberBasicSkill" type="text" placeholder="Nova perícia básica">
              <button @click="addAcademySkill('basic')">Adicionar</button>
            </div>
          </div>

          <div class="skill-type">
            <h4>Avançadas</h4>
            <div v-for="(skill, index) in character.academyData.skills.advanced" :key="'plumber-advanced-'+index" class="skill">
              <label>{{ skill.name }}:</label>
              <select v-model="skill.level">
                <option value="0">N0</option>
                <option value="1">N1</option>
                <option value="2">N2</option>
                <option value="3">N3</option>
                <option value="4">N4</option>
              </select>
              <button @click="removeAcademySkill('advanced', index)">Remover</button>
            </div>
            <div class="add-skill">
              <input v-model="newPlumberAdvancedSkill" type="text" placeholder="Nova perícia avançada">
              <button @click="addAcademySkill('advanced')">Adicionar</button>
            </div>
          </div>
        </div>

        <h3>Técnicas de Encanador</h3>
        <div class="form-group">
          <input v-model="newPlumberTechnique" type="text" placeholder="Nova técnica">
          <button @click="addAcademyTechnique">Adicionar</button>
        </div>
        <div v-for="(technique, index) in character.academyData.techniques" :key="'plumber-tech-'+index" class="technique">
          <p>{{ technique }}</p>
          <button @click="removeAcademyTechnique(index)">Remover</button>
        </div>
      </div>

      <div v-if="character.academy === 'Mágico'" class="section academy-specific">
        <h2>Mágico</h2>
        <div class="form-group">
          <label>Nível Mágico (NM):</label>
          <select v-model="character.academyData.nm">
            <option value="0">N0</option>
            <option value="1">N1</option>
            <option value="2">N2</option>
            <option value="3">N3</option>
            <option value="Mestre">Mestre</option>
          </select>
        </div>
        <div class="form-group">
          <label>Pontos de Mana (PM):</label>
          <input v-model.number="character.academyData.pm" type="number" min="0">
        </div>
        <div class="form-group">
          <label>Magias Memorizadas (MM):</label>
          <input v-model.number="character.academyData.mm" type="number" min="0">
        </div>
        
        <h3>Perícias de Mágico</h3>
        <div class="skills-section">
          <div class="skill-type">
            <h4>Básicas</h4>
            <div v-for="(skill, index) in character.academyData.skills.basic" :key="'mage-basic-'+index" class="skill">
              <label>{{ skill.name }}:</label>
              <select v-model="skill.level">
                <option value="0">N0</option>
                <option value="1">N1</option>
                <option value="2">N2</option>
                <option value="3">N3</option>
                <option value="4">N4</option>
              </select>
              <button @click="removeAcademySkill('basic', index)">Remover</button>
            </div>
            <div class="add-skill">
              <input v-model="newMageBasicSkill" type="text" placeholder="Nova perícia básica">
              <button @click="addAcademySkill('basic')">Adicionar</button>
            </div>
          </div>

          <div class="skill-type">
            <h4>Avançadas</h4>
            <div v-for="(skill, index) in character.academyData.skills.advanced" :key="'mage-advanced-'+index" class="skill">
              <label>{{ skill.name }}:</label>
              <select v-model="skill.level">
                <option value="0">N0</option>
                <option value="1">N1</option>
                <option value="2">N2</option>
                <option value="3">N3</option>
                <option value="4">N4</option>
              </select>
              <button @click="removeAcademySkill('advanced', index)">Remover</button>
            </div>
            <div class="add-skill">
              <input v-model="newMageAdvancedSkill" type="text" placeholder="Nova perícia avançada">
              <button @click="addAcademySkill('advanced')">Adicionar</button>
            </div>
          </div>
        </div>

        <h3>Técnicas de Mágico</h3>
        <div class="form-group">
          <input v-model="newMageTechnique" type="text" placeholder="Nova técnica">
          <button @click="addAcademyTechnique">Adicionar</button>
        </div>
        <div v-for="(technique, index) in character.academyData.techniques" :key="'mage-tech-'+index" class="technique">
          <p>{{ technique }}</p>
          <button @click="removeAcademyTechnique(index)">Remover</button>
        </div>

        <h3>Magias Conhecidas</h3>
        <div class="form-group">
          <textarea v-model="character.academyData.spells.known"></textarea>
        </div>
        <h4>Magias Memorizadas</h4>
        <div class="form-group">
          <textarea v-model="character.academyData.spells.memorized"></textarea>
        </div>
      </div>

      <div v-if="character.academy === 'Relojoeiro'" class="section academy-specific">
        <h2>Relojoeiro</h2>
        <div class="form-group">
          <label>Pontos Omni (PO):</label>
          <input v-model.number="character.academyData.po" type="number" min="0">
        </div>
        <div class="form-group">
          <label>Energia Omni (EO):</label>
          <input v-model.number="character.academyData.eo" type="number" min="0" max="100">
        </div>
        
        <h3>Perícias de Relojoeiro</h3>
        <div class="skills-section">
          <div class="skill-type">
            <h4>Básicas</h4>
            <div v-for="(skill, index) in character.academyData.skills.basic" :key="'watch-basic-'+index" class="skill">
              <label>{{ skill.name }}:</label>
              <select v-model="skill.level">
                <option value="0">N0</option>
                <option value="1">N1</option>
                <option value="2">N2</option>
                <option value="3">N3</option>
                <option value="4">N4</option>
              </select>
              <button @click="removeAcademySkill('basic', index)">Remover</button>
            </div>
            <div class="add-skill">
              <input v-model="newWatchBasicSkill" type="text" placeholder="Nova perícia básica">
              <button @click="addAcademySkill('basic')">Adicionar</button>
            </div>
          </div>

          <div class="skill-type">
            <h4>Avançadas</h4>
            <div v-for="(skill, index) in character.academyData.skills.advanced" :key="'watch-advanced-'+index" class="skill">
              <label>{{ skill.name }}:</label>
              <select v-model="skill.level">
                <option value="0">N0</option>
                <option value="1">N1</option>
                <option value="2">N2</option>
                <option value="3">N3</option>
                <option value="4">N4</option>
              </select>
              <button @click="removeAcademySkill('advanced', index)">Remover</button>
            </div>
            <div class="add-skill">
              <input v-model="newWatchAdvancedSkill" type="text" placeholder="Nova perícia avançada">
              <button @click="addAcademySkill('advanced')">Adicionar</button>
            </div>
          </div>
        </div>

        <h3>Técnicas de Relojoeiro</h3>
        <div class="form-group">
          <input v-model="newWatchTechnique" type="text" placeholder="Nova técnica">
          <button @click="addAcademyTechnique">Adicionar</button>
        </div>
        <div v-for="(technique, index) in character.academyData.techniques" :key="'watch-tech-'+index" class="technique">
          <p>{{ technique }}</p>
          <button @click="removeAcademyTechnique(index)">Remover</button>
        </div>

        <h3>Decatrix</h3>
        <div class="form-group">
          <label>Funções:</label>
          <textarea v-model="character.academyData.decatrix.functions"></textarea>
        </div>
        
        <h4>Lista de Aliens</h4>
        <div v-for="(alien, index) in character.academyData.decatrix.aliens" :key="'alien-'+index" class="alien">
          <div class="form-group">
            <label>Nome:</label>
            <input v-model="alien.name" type="text">
          </div>
          <div class="form-group">
            <label>Espécie:</label>
            <input v-model="alien.species" type="text">
          </div>
          <button @click="removeDecatrixAlien(index)">Remover</button>
        </div>
        <button @click="addDecatrixAlien">Adicionar Alien</button>
      </div>

      <!-- Anotações -->
      <div class="section">
        <h2>Anotações</h2>
        <div class="form-group">
          <label>Itens:</label>
          <textarea v-model="character.notes.items"></textarea>
        </div>
        <div class="form-group">
          <label>Contatos (NPCs e PJs):</label>
          <textarea v-model="character.notes.contacts"></textarea>
        </div>
        <div class="form-group">
          <label>Anotações Relevantes:</label>
          <textarea v-model="character.notes.relevant"></textarea>
        </div>
        <div class="form-group">
          <label>História:</label>
          <textarea v-model="character.notes.history"></textarea>
        </div>
      </div>

      <!-- Botões de ação -->
      <div class="action-buttons">
        <button @click="saveCharacter">Salvar Ficha</button>
        <button @click="loadCharacter">Carregar Ficha</button>
        <button @click="printCharacter">Imprimir Ficha</button>
        <button @click="resetCharacter">Nova Ficha</button>
      </div>

      <!-- Visualização da ficha -->
      <div class="character-sheet" v-if="showSheet">
        <h2>Ficha do Personagem</h2>
        
        <!-- Perfil -->
        <div class="sheet-section">
          <h3>PERFIL</h3>
          <p>|NOME: {{ character.name }} |IDADE: {{ character.age }}</p>
          <p>|ALTURA: {{ character.height }} |PESO: {{ character.weight }}</p>
          <p>|ESPÉCIE: {{ character.species }}</p>
          <p>|ACADÊMIA: {{ character.academy }}</p>
          <p>|HIERARQUIA: {{ character.hierarchy }}</p>
        </div>
        
        <!-- Status -->
        <div class="sheet-section">
          <h3>STATOS</h3>
          <p>|PV [ {{ calculatePV() }} / {{ calculatePV() }} ] |PE [ {{ calculatePE() }} / {{ calculatePE() }} ]</p>
          <p>|PA [ {{ calculatePA() }} ] |PP [ {{ calculatePP() }} ] |PC [ {{ calculatePC() }} ]</p>
        </div>
        
        <!-- Classes de Resistência -->
        <div class="sheet-section">
          <h3>CLASSES DE RESISTÊNCIA (CR)</h3>
          <p>|[{{ calculateCR('organism') }}] ORGANISMO</p>
          <p>|[{{ calculateCR('body') }}] CORPO</p>
          <p>|[{{ calculateCR('mind') }}] MENTE</p>
        </div>
        
        <!-- Atributos -->
        <div class="sheet-section">
          <h3>ATRIBUTOS ({{ calculateUsedAttributePoints() }}/{{ attributePoints }})</h3>
          <p>|FOR [ {{ character.attributes.FOR }} ] |DES [ {{ character.attributes.DES }} ]</p>
          <p>|CON [ {{ character.attributes.CON }} ] |RES [ {{ character.attributes.RES }} ]</p>
          <p>|INT [ {{ character.attributes.INT }} ] |SAB [ {{ character.attributes.SAB }} ]</p>
          <p>|CAR [ {{ character.attributes.CAR }} ] |MAL [ {{ character.attributes.MAL }} ]</p>
        </div>
        
        <!-- Poderes -->
        <div class="sheet-section" v-if="character.powers.length > 0">
          <h3>PODERES (0 a 40 a 80)</h3>
          <div v-for="(power, index) in character.powers" :key="'power-'+index">
            <p>· <strong>{{ power.name }}</strong> [ {{ power.value }} ]</p>
            <p>|HABILIDADE: {{ power.ability }}</p>
            <p>|LIMITAÇÕES: {{ power.limitations }}</p>
          </div>
        </div>
        
        <!-- Fraquezas -->
        <div class="sheet-section" v-if="character.weaknesses">
          <h3>FRAQUEZAS</h3>
          <p>{{ character.weaknesses }}</p>
        </div>
        
        <!-- Talento -->
        <div class="sheet-section" v-if="character.talent">
          <h3>TALENTO</h3>
          <p>{{ character.talent }}</p>
        </div>
        
        <!-- Perícias -->
        <div class="sheet-section">
          <h3>PERÍCIAS GERAIS</h3>
          <p>|BÁSICA</p>
          <div v-for="(skill, index) in character.skills.basic" :key="'basic-'+index">
            <p>· [N{{ skill.level }}]: {{ skill.name }}</p>
          </div>
          <p>|AVANÇADA</p>
          <div v-for="(skill, index) in character.skills.advanced" :key="'advanced-'+index">
            <p>· [N{{ skill.level }}]: {{ skill.name }}</p>
          </div>
          <p>|CIRCUNSTANCIAL</p>
          <div v-for="(skill, index) in character.skills.circumstantial" :key="'circumstantial-'+index">
            <p>· [N{{ skill.level }}]: {{ skill.name }}</p>
          </div>
        </div>
        
        <!-- Técnicas -->
        <div class="sheet-section" v-if="character.techniques.length > 0">
          <h3>TÉCNICAS GERAIS</h3>
          <div v-for="(technique, index) in character.techniques" :key="'tech-'+index">
            <p>· {{ technique }}</p>
          </div>
        </div>
        
        <!-- Anotações -->
        <div class="sheet-section">
          <h3>ANOTAÇÕES</h3>
          <p>|ITENS:</p>
          <p>{{ character.notes.items }}</p>
          <p>|CONTATOS (NPCs e PJs):</p>
          <p>{{ character.notes.contacts }}</p>
          <p>|ANOTAÇÕES (Relevantes):</p>
          <p>{{ character.notes.relevant }}</p>
          <p>|HISTÓRIA:</p>
          <p>{{ character.notes.history }}</p>
        </div>
        
        <!-- Seções específicas da academia -->
        <div v-if="character.academy === 'Adestrador'" class="sheet-section">
          <h3>FICHA ADESTRADOR</h3>
          <p>|PO [ {{ character.academyData.po }} ] Pontos omni</p>
          
          <h4>PERÍCIAS ADESTRADORES</h4>
          <p>|BÁSICA</p>
          <div v-for="(skill, index) in character.academyData.skills.basic" :key="'trainer-basic-'+index">
            <p>· [N{{ skill.level }}]: {{ skill.name }}</p>
          </div>
          <p>|AVANÇADA</p>
          <div v-for="(skill, index) in character.academyData.skills.advanced" :key="'trainer-advanced-'+index">
            <p>· [N{{ skill.level }}]: {{ skill.name }}</p>
          </div>
          
          <h4>TÉCNICAS ADESTRADORES</h4>
          <div v-for="(technique, index) in character.academyData.techniques" :key="'trainer-tech-'+index">
            <p>· {{ technique }}</p>
          </div>
          
          <h4>NEMETRIX</h4>
          <p>|FUNÇÕES</p>
          <p>{{ character.academyData.nemetrix.functions }}</p>
          <p>|LISTA DE ALIENS:</p>
          <div v-for="(alien, index) in character.academyData.nemetrix.aliens" :key="'alien-'+index">
            <p>· [{{ index + 1 }}]: {{ alien.name }} | {{ alien.species }}</p>
          </div>
        </div>
        
        <div v-if="character.academy === 'Encanador'" class="sheet-section">
          <h3>FICHA ENCANADOR</h3>
          <p>|PREPARO [ {{ character.academyData.preparation }} / {{ character.academyData.preparation }} ]</p>
          
          <h4>PERÍCIAS ENCANADORES</h4>
          <p>|BÁSICA</p>
          <div v-for="(skill, index) in character.academyData.skills.basic" :key="'plumber-basic-'+index">
            <p>· [N{{ skill.level }}]: {{ skill.name }}</p>
          </div>
          <p>|AVANÇADA</p>
          <div v-for="(skill, index) in character.academyData.skills.advanced" :key="'plumber-advanced-'+index">
            <p>· [N{{ skill.level }}]: {{ skill.name }}</p>
          </div>
          
          <h4>TÉCNICAS ENCANADORES</h4>
          <div v-for="(technique, index) in character.academyData.techniques" :key="'plumber-tech-'+index">
            <p>· {{ technique }}</p>
          </div>
        </div>
        
        <div v-if="character.academy === 'Mágico'" class="sheet-section">
          <h3>FICHA MÁGICOS</h3>
          <p>|NM [ {{ character.academyData.nm }} ]</p>
          <p>|PM [ {{ character.academyData.pm }} / {{ character.academyData.pm }} ]</p>
          <p>|MM [ {{ character.academyData.mm }} ]</p>
          
          <h4>PERÍCIAS MÁGICOS</h4>
          <p>|BÁSICA</p>
          <div v-for="(skill, index) in character.academyData.skills.basic" :key="'mage-basic-'+index">
            <p>· [N{{ skill.level }}]: {{ skill.name }}</p>
          </div>
          <p>|AVANÇADA</p>
          <div v-for="(skill, index) in character.academyData.skills.advanced" :key="'mage-advanced-'+index">
            <p>· [N{{ skill.level }}]: {{ skill.name }}</p>
          </div>
          
          <h4>TÉCNICAS MÁGICOS</h4>
          <div v-for="(technique, index) in character.academyData.techniques" :key="'mage-tech-'+index">
            <p>· {{ technique }}</p>
          </div>
          
          <h4>LISTA DE MAGIAS CONHECIDAS</h4>
          <p>{{ character.academyData.spells.known }}</p>
          <p>|MAGIAS MEMORIZADAS:</p>
          <p>{{ character.academyData.spells.memorized }}</p>
        </div>
        
        <div v-if="character.academy === 'Relojoeiro'" class="sheet-section">
          <h3>FICHA RELOJOEIRO</h3>
          <p>|PO [ {{ character.academyData.po }} ] Pontos omni</p>
          <p>|EO [ {{ character.academyData.eo }} / 100 ] Energia omni</p>
          
          <h4>PERÍCIAS RELOJOEIRO</h4>
          <p>|BÁSICA</p>
          <div v-for="(skill, index) in character.academyData.skills.basic" :key="'watch-basic-'+index">
            <p>· [N{{ skill.level }}]: {{ skill.name }}</p>
          </div>
          <p>|AVANÇADA</p>
          <div v-for="(skill, index) in character.academyData.skills.advanced" :key="'watch-advanced-'+index">
            <p>· [N{{ skill.level }}]: {{ skill.name }}</p>
          </div>
          
          <h4>TÉCNICAS RELOJOEIRO</h4>
          <div v-for="(technique, index) in character.academyData.techniques" :key="'watch-tech-'+index">
            <p>· {{ technique }}</p>
          </div>
          
          <h4>DECATRIX</h4>
          <p>|FUNÇÕES</p>
          <p>{{ character.academyData.decatrix.functions }}</p>
          <p>|LISTA DE ALIENS:</p>
          <div v-for="(alien, index) in character.academyData.decatrix.aliens" :key="'decatrix-alien-'+index">
            <p>· [{{ index + 1 }}]: {{ alien.name }} | {{ alien.species }}</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      showSheet: false,
      attributePoints: 24,
      newPower: {
        name: '',
        value: 0,
        ability: '',
        limitations: ''
      },
      newBasicSkill: '',
      newAdvancedSkill: '',
      newCircumstantialSkill: '',
      newTechnique: '',
      newTrainerBasicSkill: '',
      newTrainerAdvancedSkill: '',
      newTrainerTechnique: '',
      newPlumberBasicSkill: '',
      newPlumberAdvancedSkill: '',
      newPlumberTechnique: '',
      newMageBasicSkill: '',
      newMageAdvancedSkill: '',
      newMageTechnique: '',
      newWatchBasicSkill: '',
      newWatchAdvancedSkill: '',
      newWatchTechnique: '',
      speciesList: [
        'Aerofibiano', 'Anodita', 'Antigravitesla', 'Arachnachimp', 'Argit', 'Amperis', 'Apoplixianos', 
        'Atômico', 'Atrocianos', 'Basalt', 'Bigotoides', 'Biosovortianos', 'Bob The Blob', 'Calbreus', 
        'Cascanos', 'Cerebrocrustaceanos', 'Cephalod-Ae', 'Chimeras Sui Generis', 'Citrakayahs', 
        'Cocorocoide', 'Conductoides', 'Chronosapianos', 'Churls', 'Crystalsapiens', 'Dragão Alien', 
        'Detrovites', 'Ectonuritas', 'Ekoplektoid', 'Escaravola', 'Espantóide', 'Floraunas', 'Fulminis', 
        'Galileanos', 'Galvanianos', 'Gimlinopithecus', 'Geochelone Aerios', 'Gourmandos', 'Humanos', 
        'Humanos de Ledgerdomain', 'Inkursianos', 'InkthiPerambuloides', 'Kinecelerandos', 'Kraahos', 
        'Lenopanos', 'Lepidoptereanos', 'Lewodanianos', 'Loboans', 'Mecamorfos Galvanics', 
        'Merlini Sapiens', 'Metanosianos', 'Muroids', 'Nanochip Humano', 'Necrofriggianos', 'Nemuninas', 
        'Nosedeenianos', 'Opiticoides', 'Orishanos', 'Orthopterranos', 'Oryctinis', 'Pantophages', 
        'Pelarotas Arburianos', 'Petrosapiens', 'Pisccis Premann', 'Pisccis Volanns', 'Pitorovosaurianos', 
        'Planchaküles', 'Podrão', 'Polar Manzardills', 'Polimorfos', 'Protosts', 'Prypiatosians B', 
        'Pugnivoros', 'Pyronitas', 'Revonnah Ganders', 'Sapiens Celestiais', 'Segmento Sapiens', 
        'Snidellerodontu', 'Soberanos', 'Solinoides', 'Sonorosianos', 'Spheroides', 'Splixions', 
        'Sylonnoids', 'Talpaedanos', 'Tetramandos', 'Thalassiano', 'Thep Khufans', 'To,Kustares', 
        'Transylianos', 'Últimos', 'Uxorites', 'Vaxasaurianos', 'Vladats', 'Vreedls', 'Vulpimanceres', 
        'Xerges', 'Zaroffianos'
      ],
      character: {
        name: '',
        age: '',
        height: '',
        weight: '',
        species: '',
        academy: 'Adestrador',
        hierarchy: 'Cadete',
        talent: '',
        attributes: {
          FOR: 0,
          DES: 0,
          CON: 0,
          RES: 0,
          INT: 0,
          SAB: 0,
          CAR: 0,
          MAL: 0
        },
        powers: [],
        weaknesses: '',
        skills: {
          basic: [],
          advanced: [],
          circumstantial: []
        },
        techniques: [],
        notes: {
          items: '',
          contacts: '',
          relevant: '',
          history: ''
        },
        academyData: {
          // Adestrador
          po: 50,
          skills: {
            basic: [],
            advanced: []
          },
          techniques: [],
          nemetrix: {
            functions: 'FUNÇÕES BÁSICAS [...]',
            aliens: []
          },
          // Encanador
          preparation: 5,
          // Mágico
          nm: 0,
          pm: 0,
          mm: 0,
          spells: {
            known: '',
            memorized: ''
          },
          // Relojoeiro
          eo: 100,
          decatrix: {
            functions: 'FUNÇÕES BÁSICAS [...]',
            aliens: []
          }
        }
      }
    }
  },
  methods: {
    updateAttributePoints() {
      switch(this.character.hierarchy) {
        case 'Cadete':
          this.attributePoints = 24;
          break;
        case 'Cadete Oficializado':
          this.attributePoints = 34;
          break;
        case 'Oficial':
          this.attributePoints = 44;
          break;
        default:
          this.attributePoints = 24;
      }
    },
    getSpeciesBonus() {
      // Implementar lógica para bônus racial baseado na espécie
      // Por enquanto retorna 0 para todas as espécies
      return 0;
    },
    addPower() {
      if (this.newPower.name) {
        this.character.powers.push({
          name: this.newPower.name,
          value: this.newPower.value,
          ability: this.newPower.ability,
          limitations: this.newPower.limitations
        });
        this.newPower = {
          name: '',
          value: 0,
          ability: '',
          limitations: ''
        };
      }
    },
    removePower(index) {
      this.character.powers.splice(index, 1);
    },
    addSkill(type) {
      let newSkillName = '';
      switch(type) {
        case 'basic':
          newSkillName = this.newBasicSkill;
          break;
        case 'advanced':
          newSkillName = this.newAdvancedSkill;
          break;
        case 'circumstantial':
          newSkillName = this.newCircumstantialSkill;
          break;
      }
      
      if (newSkillName) {
        this.character.skills[type].push({
          name: newSkillName,
          level: 0
        });
        
        switch(type) {
          case 'basic':
            this.newBasicSkill = '';
            break;
          case 'advanced':
            this.newAdvancedSkill = '';
            break;
          case 'circumstantial':
            this.newCircumstantialSkill = '';
            break;
        }
      }
    },
    removeSkill(type, index) {
      this.character.skills[type].splice(index, 1);
    },
    addTechnique() {
      if (this.newTechnique) {
        this.character.techniques.push(this.newTechnique);
        this.newTechnique = '';
      }
    },
    removeTechnique(index) {
      this.character.techniques.splice(index, 1);
    },
    addAcademySkill(type) {
      let newSkillName = '';
      let newSkillVar = '';
      
      switch(this.character.academy) {
        case 'Adestrador':
          if (type === 'basic') {
            newSkillName = this.newTrainerBasicSkill;
            newSkillVar = 'newTrainerBasicSkill';
          } else {
            newSkillName = this.newTrainerAdvancedSkill;
            newSkillVar = 'newTrainerAdvancedSkill';
          }
          break;
        case 'Encanador':
          if (type === 'basic') {
            newSkillName = this.newPlumberBasicSkill;
            newSkillVar = 'newPlumberBasicSkill';
          } else {
            newSkillName = this.newPlumberAdvancedSkill;
            newSkillVar = 'newPlumberAdvancedSkill';
          }
          break;
        case 'Mágico':
          if (type === 'basic') {
            newSkillName = this.newMageBasicSkill;
            newSkillVar = 'newMageBasicSkill';
          } else {
            newSkillName = this.newMageAdvancedSkill;
            newSkillVar = 'newMageAdvancedSkill';
          }
          break;
        case 'Relojoeiro':
          if (type === 'basic') {
            newSkillName = this.newWatchBasicSkill;
            newSkillVar = 'newWatchBasicSkill';
          } else {
            newSkillName = this.newWatchAdvancedSkill;
            newSkillVar = 'newWatchAdvancedSkill';
          }
          break;
      }
      
      if (newSkillName) {
        this.character.academyData.skills[type].push({
          name: newSkillName,
          level: 0
        });
        this[newSkillVar] = '';
      }
    },
    removeAcademySkill(type, index) {
      this.character.academyData.skills[type].splice(index, 1);
    },
    addAcademyTechnique() {
      let newTechnique = '';
      let newTechniqueVar = '';
      
      switch(this.character.academy) {
        case 'Adestrador':
          newTechnique = this.newTrainerTechnique;
          newTechniqueVar = 'newTrainerTechnique';
          break;
        case 'Encanador':
          newTechnique = this.newPlumberTechnique;
          newTechniqueVar = 'newPlumberTechnique';
          break;
        case 'Mágico':
          newTechnique = this.newMageTechnique;
          newTechniqueVar = 'newMageTechnique';
          break;
        case 'Relojoeiro':
          newTechnique = this.newWatchTechnique;
          newTechniqueVar = 'newWatchTechnique';
          break;
      }
      
      if (newTechnique) {
        this.character.academyData.techniques.push(newTechnique);
        this[newTechniqueVar] = '';
      }
    },
    removeAcademyTechnique(index) {
      this.character.academyData.techniques.splice(index, 1);
    },
    addAlien() {
      this.character.academyData.nemetrix.aliens.push({
        name: '',
        species: ''
      });
    },
    removeAlien(index) {
      this.character.academyData.nemetrix.aliens.splice(index, 1);
    },
    addDecatrixAlien() {
      this.character.academyData.decatrix.aliens.push({
        name: '',
        species: ''
      });
    },
    removeDecatrixAlien(index) {
      this.character.academyData.decatrix.aliens.splice(index, 1);
    },
    updateAcademySpecifics() {
      // Reset academy data when changing academy
      this.character.academyData = {
        // Adestrador
        po: 50,
        skills: {
          basic: [],
          advanced: []
        },
        techniques: [],
        nemetrix: {
          functions: 'FUNÇÕES BÁSICAS [...]',
          aliens: []
        },
        // Encanador
        preparation: 5,
        // Mágico
        nm: 0,
        pm: 0,
        mm: 0,
        spells: {
          known: '',
          memorized: ''
        },
        // Relojoeiro
        eo: 100,
        decatrix: {
          functions: 'FUNÇÕES BÁSICAS [...]',
          aliens: []
        }
      };
    },
    calculatePV() {
      return (this.character.attributes.CON * 5) + 25;
    },
    calculatePE() {
      return Math.floor((this.character.attributes.FOR + this.character.attributes.DES + 
                        this.character.attributes.CON + this.character.attributes.RES) / 4);
    },
    calculatePA() {
      return Math.floor((this.character.attributes.DES + this.character.attributes.SAB) / 20) + 1;
    },
    calculatePP() {
      return 10 + this.character.attributes.SAB;
    },
    calculatePC() {
      return 10 + this.character.attributes.CAR;
    },
    calculateCR(type) {
      switch(type) {
        case 'organism':
          return this.character.attributes.CON + 15;
        case 'body':
          return Math.floor((this.character.attributes.FOR + this.character.attributes.RES) / 2) + 10;
        case 'mind':
          return Math.floor((this.character.attributes.SAB + this.character.attributes.INT) / 2) + 10;
        default:
          return 0;
      }
    },
    calculateUsedAttributePoints() {
      return Object.values(this.character.attributes).reduce((sum, value) => sum + value, 0);
    },
    saveCharacter() {
      const dataStr = JSON.stringify(this.character);
      localStorage.setItem('ben10RPGCharacter', dataStr);
      alert('Ficha salva com sucesso!');
    },
    loadCharacter() {
      const savedCharacter = localStorage.getItem('ben10RPGCharacter');
      if (savedCharacter) {
        this.character = JSON.parse(savedCharacter);
        alert('Ficha carregada com sucesso!');
      } else {
        alert('Nenhuma ficha encontrada para carregar.');
      }
    },
    printCharacter() {
      this.showSheet = true;
      setTimeout(() => {
        window.print();
        this.showSheet = false;
      }, 100);
    },
    resetCharacter() {
      if (confirm('Tem certeza que deseja criar uma nova ficha? Todos os dados não salvos serão perdidos.')) {
        this.character = {
          name: '',
          age: '',
          height: '',
          weight: '',
          species: '',
          academy: 'Adestrador',
          hierarchy: 'Cadete',
          talent: '',
          attributes: {
            FOR: 0,
            DES: 0,
            CON: 0,
            RES: 0,
            INT: 0,
            SAB: 0,
            CAR: 0,
            MAL: 0
          },
          powers: [],
          weaknesses: '',
          skills: {
            basic: [],
            advanced: [],
            circumstantial: []
          },
          techniques: [],
          notes: {
            items: '',
            contacts: '',
            relevant: '',
            history: ''
          },
          academyData: {
            po: 50,
            skills: {
              basic: [],
              advanced: []
            },
            techniques: [],
            nemetrix: {
              functions: 'FUNÇÕES BÁSICAS [...]',
              aliens: []
            },
            preparation: 5,
            nm: 0,
            pm: 0,
            mm: 0,
            spells: {
              known: '',
              memorized: ''
            },
            eo: 100,
            decatrix: {
              functions: 'FUNÇÕES BÁSICAS [...]',
              aliens: []
            }
          }
        };
        this.attributePoints = 24;
      }
    }
  },
  created() {
    this.updateAttributePoints();
  }
}
</script>

<style>
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  font-family: Arial, sans-serif;
  line-height: 1.6;
  color: #333;
  background-color: #f5f5f5;
}

#app {
  padding: 20px;
}

.container {
  max-width: 1200px;
  margin: 0 auto;
  background-color: white;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}

h1, h2, h3, h4 {
  margin-bottom: 15px;
  color: #2c3e50;
}

.section {
  margin-bottom: 30px;
  padding: 20px;
  border: 1px solid #ddd;
  border-radius: 5px;
  background-color: #f9f9f9;
}

.form-group {
  margin-bottom: 15px;
}

.form-group label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
}

.form-group input[type="text"],
.form-group input[type="number"],
.form-group select,
.form-group textarea {
  width: 100%;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.form-group textarea {
  min-height: 100px;
  resize: vertical;
}

button {
  padding: 8px 15px;
  background-color: #3498db;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  margin-right: 10px;
  margin-bottom: 10px;
}

button:hover {
  background-color: #2980b9;
}

.attributes-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 15px;
}

.attribute {
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  background-color: white;
}

.skills-section {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 15px;
}

.skill-type {
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  background-color: white;
}

.skill {
  margin-bottom: 10px;
  padding-bottom: 10px;
  border-bottom: 1px solid #eee;
}

.skill:last-child {
  border-bottom: none;
}

.add-skill {
  margin-top: 15px;
}

.power, .technique, .alien {
  margin-bottom: 15px;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  background-color: white;
}

.action-buttons {
  margin-top: 30px;
  text-align: center;
}

/* Character sheet print styles */
.character-sheet {
  display: none;
}

@media print {
  body * {
    visibility: hidden;
  }
  .character-sheet, .character-sheet * {
    visibility: visible;
  }
  .character-sheet {
    position: absolute;
    left: 0;
    top: 0;
    width: 100%;
    display: block;
  }
  .sheet-section {
    margin-bottom: 20px;
    page-break-inside: avoid;
  }
  button {
    display: none;
  }
}

/* Responsive adjustments */
@media (max-width: 768px) {
  .attributes-grid, .skills-section {
    grid-template-columns: 1fr;
  }
}
</style>