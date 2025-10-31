<script setup>
import { ref, computed } from 'vue';

import pumpkinImage from '@/assets/fantome.png'; 
import ghostImage from '@/assets/sorciere.png';
import hatImage from '@/assets/halloween.png';
import spiderImage from '@/assets/mort.png';
import vampireImage from '@/assets/vampire.png';
import tombeImage from '@/assets/tombe.png';

const TOUTES_LES_PAIRES = [
    { valeur: 'A', image: pumpkinImage },
    { valeur: 'B', image: ghostImage },
    { valeur: 'C', image: hatImage },
    { valeur: 'D', image: spiderImage },
    { valeur: 'E', image: vampireImage },
    { valeur: 'F', image: tombeImage }
];

function melangeSimple(tableau) {
  const tableauMelange = [...tableau]; 
  tableauMelange.sort(() => Math.random() - 0.5); 
  return tableauMelange;
}

const cartes = ref([]);
const cartesRetournees = ref([]);
const verrouillePlateau = ref(false);
const difficulteActuelle = ref('');
const coups = ref(0);

const reinitialiserCartesRetournees = () => {
    cartesRetournees.value = [];
    verrouillePlateau.value = false;
};

const verifierPaire = () => {
    const [carte1, carte2] = cartesRetournees.value;
    coups.value++;

    if (carte1.valeur === carte2.valeur) {
        carte1.estTrouvee = true;
        carte2.estTrouvee = true;
        reinitialiserCartesRetournees();
    } else {
        setTimeout(() => {
            carte1.estRetournee = false;
            carte2.estRetournee = false;
            reinitialiserCartesRetournees();
        }, 1000); 
    }
};

const retournerCarte = (carte) => {
    if (verrouillePlateau.value || carte.estRetournee || carte.estTrouvee) {
        return;
    }
    
    carte.estRetournee = true;
    cartesRetournees.value.push(carte);

    if (cartesRetournees.value.length === 2) {
        verrouillePlateau.value = true;
        verifierPaire();
    }
};

const definirDifficulte = (difficulte) => {
    difficulteActuelle.value = difficulte;
    
    let comptePaire = 0;
    switch (difficulte) {
        case 'facile': comptePaire = 2; break; 
        case 'intermediaire': comptePaire = 4; break;
        case 'difficile': default: comptePaire = 6; break;
    }

    const pairesSelectionnees = TOUTES_LES_PAIRES.slice(0, comptePaire);
    let nouvellesCartes = [];
    pairesSelectionnees.forEach(paire => {
        nouvellesCartes.push({ ...paire, estRetournee: false, estTrouvee: false }); 
        nouvellesCartes.push({ ...paire, estRetournee: false, estTrouvee: false });
    });
    
    const cartesMelangees = melangeSimple(nouvellesCartes);
    
    cartes.value = cartesMelangees.map((carte, index) => ({
        ...carte,
        idUnique: index,
    }));

    coups.value = 0;
    verrouillePlateau.value = false;
    cartesRetournees.value = [];
};

const recommencerJeu = () => {
    definirDifficulte(difficulteActuelle.value);
};

const styleGrille = computed(() => {
    let cols = 2;
    if (difficulteActuelle.value === 'intermediaire' || difficulteActuelle.value === 'difficile') {
        cols = 4;
    }
    return `grid-template-columns: repeat(${cols}, 100px);`;
});

definirDifficulte('intermediaire');
</script>

<template>
    <div class="conteneur-jeu">
        <h1>Memory Game : Halloween</h1>
        
        <h2>Choisir le niveau de difficulté :
            <button 
                @click="definirDifficulte('facile')" 
                :class="{ active: difficulteActuelle === 'facile' }"
            >
                Facile (4)
            </button>
            <button 
                @click="definirDifficulte('intermediaire')" 
                :class="{ active: difficulteActuelle === 'intermediaire' }"
            >
                Intermédiaire (8)
            </button>
            <button 
                @click="definirDifficulte('difficile')" 
                :class="{ active: difficulteActuelle === 'difficile' }"
            >
                Difficile (12)
            </button>
        </h2>

        <div class="stats-jeu">
            <p>Coups : **{{ coups }}**</p>
            <button v-if="difficulteActuelle" @click="recommencerJeu" class="bouton-recommencer">
                Recommencer
            </button>
        </div>

        <div v-if="cartes.length > 0" class="plateau-jeu" :style="styleGrille"> 
            
            <div 
                v-for="carte in cartes" 
                :key="carte.idUnique" 
                class="carte"
                :class="{ 'est-trouvee': carte.estTrouvee, 'est-retournee': carte.estRetournee || carte.estTrouvee }"
                @click="retournerCarte(carte)"
            >
                <div class="face-interne">
                    <img :src="carte.image" class="face-visible" alt="Face de la carte" />
                    <span class="dos-carte">?</span>
                </div>
            </div>
        </div>
        <div v-else>
            <p style="text-align: center; font-size: 1.2em;">Veuillez choisir un niveau pour commencer la partie.</p>
        </div>
        
        <h2 v-if="cartes.length > 0 && cartes.every(c => c.estTrouvee)" style="text-align: center; color: green;">🎉 C'est Gagné !</h2>
    </div>
</template>

<style scoped>

.difficulty-selector button {
    padding: 8px 15px; margin: 0 5px; background-color: #ecf0f1; border: 1px solid #ccc; cursor: pointer; border-radius: 4px; transition: background-color 0.2s;
}
.difficulty-selector button.active {
    background-color: #3498db; color: white; font-weight: bold; border-color: #3498db;
}


.plateau-jeu {
    display: grid;
    gap: 10px;
    margin: 20px auto;
    width: fit-content;
    padding: 20px;
    border: 1px solid #eee;
    border-radius: 8px;
    perspective: 1000px; 
}


.carte {
    width: 100px;
    height: 100px;
    position: relative;
    cursor: pointer;
    transform-style: preserve-3d;
    transition: transform 0.6s;
    border-radius: 5px;
}


.carte.est-retournee, .carte.est-trouvee { 
    transform: rotateY(180deg);
}

.face-interne {
    position: absolute;
    width: 100%;
    height: 100%;
    transform-style: preserve-3d;
}

.face-visible, .dos-carte {
    position: absolute;
    width: 100%;
    height: 100%;
    backface-visibility: hidden;
    border-radius: 5px;
    display: flex;
    justify-content: center;
    align-items: center;
}

.face-visible {
    transform: rotateY(180deg); 
    padding: 0;
}

.face-visible img {
    width: 100%;
    height: 100%;
    object-fit: cover;
    border-radius: 5px;
}

.dos-carte {
    background-color: #3498db;
    color: white;
    font-size: 2em;
    border: 2px solid #3498db;
}

.carte.est-trouvee {
    border-color: #2ecc71; 
    opacity: 0.7; 
    cursor: default;
}


.stats-jeu {
    display: flex;
    justify-content: space-around;
    align-items: center;
    margin-bottom: 20px;
    background: #f7f7f7;
    padding: 10px;
    border-radius: 8px;
}

.bouton-recommencer {
    padding: 8px 15px;
    background-color: #e67e22; 
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
}


h1 {
    text-align: center;
    font-weight: bold;
}
h2 {
    font-size: 16px;
    text-align: center;
    margin-top: 20px;
    margin-bottom: 20px;
}
button{
  background-color: #2ecc71;
  border-radius: 5px; 
  padding: 5px;
  margin: 5px;
  border: none;
  cursor: pointer;
}
</style>