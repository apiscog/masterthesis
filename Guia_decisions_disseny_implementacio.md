# Guia provisional de decisions de disseny i implementació

## Finalitat del document

Aquest document resumeix el camí tècnic que es desprèn de la proposta i de
l'estat de l'art del TFM. La seva funció és servir com a guia per a les
iteracions següents i facilitar que cada decisió d'implementació mantingui una
relació clara amb el problema que es vol estudiar.

No és una especificació tancada. Durant el desenvolupament es podran modificar
algoritmes, tecnologies, mètriques o límits si les proves mostren que una altra
opció és més adequada. Quan això passi, caldrà registrar el canvi i justificar
com afecta els objectius i els experiments.

La proposta aprovada, els enunciats de cada entrega i la memòria del TFM tenen
prioritat sobre aquesta guia. Si hi ha una contradicció, primer s'haurà de
revisar la guia i adaptar-la a la font acadèmica corresponent.

## Criteri per interpretar les decisions

- **Base acordada**: forma part del plantejament actual del TFM i orienta la
  primera implementació.
- **Decisió prevista**: és l'opció preferida, però encara s'ha de confirmar
  durant el disseny o amb una prova tècnica.
- **Decisió pendent**: necessita una comparació, una fórmula o un criteri que
  encara no s'ha concretat.
- **Opcional**: només s'abordarà si el nucli del projecte està complet i el
  calendari ho permet.

## 1. Problema que representarà la plataforma

- **Base acordada**: la plataforma estudiarà com es planifiquen tasques
  asíncrones quan diverses tasques competeixen per una capacitat limitada.
- **Base acordada**: els experiments utilitzaran dades i escenaris sintètics.
- **Base acordada**: no hi haurà connexions amb empreses, botigues ni sistemes
  externs reals.
- **Base acordada**: una botiga en línia fictícia podrà donar context als
  exemples, però el model continuarà sent aplicable a altres sistemes backend.
- **Base acordada**: la plataforma permetrà comparar comportaments dins del
  model. Per tant, no garantirà que una política millori qualsevol sistema real.

## 2. Model inicial de les tasques i els recursos

- **Base acordada**: cada tasca tindrà, com a mínim, un identificador, un tipus,
  un instant d'arribada, una durada estimada, una prioritat i un estat.
- **Base acordada**: la capacitat es representarà mitjançant un nombre
  configurable de processos de treball o *workers*.
- **Base acordada**: el model serà no preemptiu. Una tasca iniciada conservarà
  el worker fins que finalitzi i no serà interrompuda per una tasca més urgent.
- **Base acordada**: la decisió de planificació serà dinàmica i es prendrà quan
  hi hagi capacitat lliure.
- **Decisió prevista**: l'ordre d'arribada, l'ordre d'inici i l'ordre de
  finalització es conservaran com a dades diferents.
- **Decisió pendent**: cal concretar els estats exactes de la tasca i les
  transicions permeses entre aquests estats.
- **Opcional**: es podran afegir fallades, intents i reintents amb *backoff* quan
  el flux principal estigui verificat.

## 3. Motor de simulació

- **Base acordada**: es desenvoluparà un motor propi de simulació
  d'esdeveniments discrets.
- **Base acordada**: el motor utilitzarà un rellotge virtual. Això permetrà
  avançar fins al següent esdeveniment sense esperar que passi el temps real.
- **Base acordada**: el motor representarà arribades, inicis, finalitzacions,
  cues i assignacions de workers.
- **Base acordada**: no reproduirà protocols de missatgeria, replicació,
  confirmació ni altres garanties internes d'un broker productiu.
- **Decisió prevista**: una cua d'esdeveniments ordenada per temps simulat
  dirigirà l'avanç del rellotge.
- **Decisió pendent**: cal definir l'ordre de desempat quan diversos
  esdeveniments comparteixen el mateix instant.
- **Decisió pendent**: cal establir la condició de finalització d'una simulació,
  per exemple quan no quedin tasques ni esdeveniments pendents.

## 4. Polítiques de planificació

- **Base acordada**: FIFO serà la línia base i seleccionarà les tasques segons
  l'ordre d'arribada.
- **Base acordada**: la prioritat estricta seleccionarà primer les tasques amb
  una prioritat superior.
- **Base acordada**: la prioritat amb envelliment augmentarà la prioritat
  efectiva de les tasques a mesura que acumulin temps d'espera.
- **Decisió prevista**: les polítiques implementaran una interfície comuna.
  D'aquesta manera, es podran intercanviar sense modificar el motor.
- **Decisió prevista**: FIFO actuarà com a criteri de desempat entre tasques amb
  la mateixa prioritat efectiva.
- **Decisió pendent**: cal definir la fórmula d'envelliment, els pesos, els
  intervals d'actualització i els possibles límits.
- **Decisió pendent**: cal decidir si la prioritat es representarà amb nombres
  ascendents o descendents i mantenir aquest criteri a tota l'aplicació.
- **Criteri d'avaluació**: no es considerarà que una política és millor de
  manera general. S'analitzarà en quins escenaris ofereix avantatges o
  inconvenients.

## 5. Mètriques que es volen calcular

- **Base acordada**: temps mitjà, mediana, percentil 95 i temps màxim d'espera.
- **Base acordada**: temps total de cada tasca dins del sistema.
- **Base acordada**: nombre i percentatge de tasques completades.
- **Base acordada**: *throughput*, entès com les tasques completades per unitat
  de temps simulat.
- **Base acordada**: utilització dels workers.
- **Base acordada**: evolució de la longitud de la cua.
- **Base acordada**: resultats separats per tipus i prioritat de tasca.
- **Decisió prevista**: es calcularà el *makespan*, que és el temps fins que
  finalitza l'última tasca d'un escenari finit.
- **Decisió prevista**: es comptaran les tasques que superin un llindar
  d'espera per detectar esperes extremes.
- **Decisió pendent**: cal establir el llindar d'espera i justificar-lo segons
  cada escenari.
- **Opcional**: si s'implementen fallades, també es mesuraran els intents i els
  reintents.

## 6. Disseny dels experiments

- **Base acordada**: totes les polítiques rebran la mateixa llista de tasques i
  la mateixa configuració de recursos.
- **Base acordada**: els escenaris aleatoris conservaran la llavor utilitzada
  per poder generar de nou la mateixa entrada.
- **Base acordada**: es conservaran els paràmetres exactes de cada execució.
- **Base acordada**: hi haurà casos petits amb resultats calculables manualment
  per verificar el motor.
- **Base acordada**: s'estudiaran, com a mínim, situacions de càrrega normal,
  pics de demanda, recursos escassos i combinacions de prioritats.
- **Decisió prevista**: els experiments aleatoris es repetiran diverses vegades
  i se'n resumiran els resultats.
- **Decisió pendent**: cal definir el nombre de repeticions i el criteri per
  considerar que els resultats són prou estables.
- **Decisió pendent**: cal concretar les distribucions d'arribada i de durada de
  les tasques.
- **Criteri metodològic**: se seguiran les directrius STRESS com a orientació
  per documentar els objectius, el model, les dades i els experiments.

## 7. Arquitectura prevista

La separació inicial de responsabilitats serà la següent:

| Component | Responsabilitat principal | Orientació actual |
|---|---|---|
| Interfície web | Configurar escenaris i interpretar resultats | Frontend amb React, una biblioteca d'interfícies, i TypeScript |
| API | Rebre peticions i coordinar els casos d'ús | Backend amb Node.js, TypeScript i previsiblement el framework NestJS |
| Motor | Executar el rellotge virtual i les polítiques | Mòdul propi i independent de la interfície |
| Persistència | Conservar escenaris, execucions i mètriques | Base de dades relacional PostgreSQL |
| Entorn local | Executar els components de manera reproduïble | Contenidors coordinats amb Docker i Docker Compose |

- **Base acordada**: el motor quedarà separat de l'API, la base de dades i la
  interfície web.
- **Base acordada**: la interfície no requerirà consultar logs ni taules
  internes per entendre els resultats.
- **Decisió prevista**: el backend seguirà una estructura modular que permeti
  provar el domini sense iniciar tota l'aplicació.
- **Decisió prevista**: els escenaris, les execucions i els resultats tindran
  identificadors propis.
- **Decisió pendent**: cal concretar els mòduls, els casos d'ús i els contractes
  de l'API abans d'implementar els controladors.
- **Decisió pendent**: cal seleccionar la biblioteca de gràfics del frontend.

## 8. Persistència i reproduïbilitat

- **Base acordada**: es conservaran els escenaris i els seus paràmetres.
- **Base acordada**: cada execució registrarà la política, la llavor, els
  recursos disponibles i la versió del model.
- **Base acordada**: es conservaran les mètriques necessàries per comparar les
  execucions.
- **Decisió prevista**: una comparació agruparà diverses execucions del mateix
  escenari, una per cada política seleccionada.
- **Decisió pendent**: cal decidir si es persisteixen tots els esdeveniments o
  només les dades agregades i les traces necessàries per al diagnòstic.
- **Decisió pendent**: cal definir com s'identificarà la versió del model i de
  la configuració experimental.

## 9. Verificació i proves

- **Base acordada**: hi haurà proves unitàries del motor, de cada política i de
  les transicions d'estat.
- **Base acordada**: es prepararan proves d'integració per als fluxos principals
  de l'API i la persistència.
- **Base acordada**: es comprovaran invariants del model. Per exemple, una tasca
  no podrà començar abans d'arribar i un worker no podrà executar dues tasques
  alhora.
- **Decisió prevista**: les proves de les polítiques reutilitzaran els mateixos
  escenaris petits per facilitar-ne la comparació.
- **Decisió prevista**: les traces d'execució ajudaran a diagnosticar errors del
  motor, però no substituiran les proves automatitzades.
- **Opcional**: les proves de rendiment del simulador només s'afegiran si
  responen una pregunta concreta del TFM.

## 10. Relació amb eines existents

- **Base acordada**: BullMQ i RabbitMQ serveixen com a referències de cues
  operatives, però no substituiran el motor de simulació.
- **Base acordada**: SimPy, GridSim i CloudSim aporten referències sobre temps
  virtual, esdeveniments i experimentació, però no són la plataforma que es vol
  construir.
- **Base acordada**: n8n és una referència d'interfície web i execució de fluxos
  reals, però no proporciona la comparació experimental prevista.
- **Opcional**: BullMQ i Redis es podran utilitzar en una demostració
  complementària quan el motor propi estigui acabat i verificat.

## 11. Ordre d'implementació recomanat

1. Definir el vocabulari del domini, els tipus de dades i els estats de les
   tasques.
2. Especificar els esdeveniments, el rellotge virtual i la condició de
   finalització.
3. Implementar un motor mínim amb una sola cua FIFO i un worker.
4. Verificar el motor amb casos petits i invariants.
5. Generalitzar el nombre de workers i afegir la interfície comuna de
   polítiques.
6. Implementar la prioritat estricta i la prioritat amb envelliment.
7. Calcular les mètriques bàsiques i contrastar-les amb casos coneguts.
8. Definir els contractes de l'API i la persistència dels experiments.
9. Construir el flux web per crear un escenari, executar la comparació i
   consultar els resultats.
10. Preparar els escenaris experimentals, les repeticions i les visualitzacions.
11. Documentar les decisions finals, els canvis respecte d'aquesta guia i les
    limitacions observades.

## 12. Properes decisions que convé tancar

- [ ] Model d'estats i transicions de les tasques.
- [ ] Regla de desempat entre esdeveniments simultanis.
- [ ] Representació i sentit numèric de la prioritat.
- [ ] Fórmula i paràmetres de l'envelliment.
- [ ] Distribucions d'arribada i durada dels escenaris.
- [ ] Llindar utilitzat per identificar esperes extremes.
- [ ] Nombre de repeticions dels experiments aleatoris.
- [ ] Estructura dels mòduls del backend i contractes de l'API.
- [ ] Model de dades de PostgreSQL i nivell de detall de les traces.
- [ ] Biblioteca de gràfics i representacions principals del frontend.
- [ ] Criteri d'acceptació de la primera versió funcional.

## 13. Regla per actualitzar aquesta guia

Quan una decisió canviï, s'ha d'actualitzar el punt corresponent i registrar:

- quina decisió anterior es modifica
- quin problema o evidència motiva el canvi
- quina alternativa s'adopta
- com afecta l'abast, les proves, les mètriques o la planificació

D'aquesta manera, el document actuarà com un mapa del projecte i també com un
registre breu de l'evolució del disseny.
