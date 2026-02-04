## Capito! Miglioriamo l'Automazione

I problemi che hai identificato:
1. **Non triggera quando l'illuminazione scende** (ma la presenza è già attiva)
2. **Il delay non si cancella** se esci e rientri rapidamente

---

## ✅ Automazione Migliorata

```yaml
alias: Presenza Salotto Migliorata
description: Luce automatica basata su presenza E illuminazione

triggers:
  # Trigger 1: Cambio presenza
  - entity_id: binary_sensor.presenza_camera_zone_1_occupancy
    trigger: state
    id: presence_change
    
  # Trigger 2: Illuminazione scende sotto soglia (NUOVO!)
  - entity_id: sensor.presenza_camera_illuminance
    trigger: numeric_state
    below: 10
    id: illuminance_low

conditions: []

actions:
  - choose:
      # === ACCENDI LUCE ===
      - conditions:
          - condition: state
            entity_id: binary_sensor.presenza_camera_zone_1_occupancy
            state: "on"
          - condition: numeric_state
            entity_id: sensor.presenza_camera_illuminance
            below: 10
        sequence:
          - target:
              entity_id: light.luce_salotto
            action: light.turn_on
            
      # === SPEGNI LUCE (con delay riavviabile) ===
      - conditions:
          - condition: state
            entity_id: binary_sensor.presenza_camera_zone_1_occupancy
            state: "off"
        sequence:
          # Wait fino a 2 minuti MA si interrompe se torna presenza
          - wait_for_trigger:
              - entity_id: binary_sensor.presenza_camera_zone_1_occupancy
                trigger: state
                to: "on"
            timeout: "00:02:00"
            continue_on_timeout: true
            
          # Se non è tornato nessuno (timeout scaduto), spegni
          - condition: state
            entity_id: binary_sensor.presenza_camera_zone_1_occupancy
            state: "off"
          - target:
              entity_id: light.luce_salotto
            action: light.turn_off

mode: restart  # IMPORTANTE: se esci e rientri, il delay riparte
```

---

## 🔄 Come Funziona

| Scenario | Comportamento |
|----------|---------------|
| Entri nella stanza buia | ✅ Luce si accende |
| Sei già dentro e cala il sole | ✅ Luce si accende (trigger illuminance!) |
| Esci dalla stanza | ⏱️ Aspetta 2 min, poi spegne |
| Esci, torni dopo 1 min, poi esci | 🔄 Il timer **riparte** da 2 min |
| Esci e torni dopo 30 sec | ❌ Luce resta accesa |

---

## 💡 Alternative Avanzate

Se vuoi anche **spegnere la luce quando c'è abbastanza luce naturale** (anche se sei presente):

```yaml
# Aggiungi questo trigger
- entity_id: sensor.presenza_camera_illuminance
  trigger: numeric_state
  above: 50  # Soglia luce sufficiente
  id: illuminance_high

# E aggiungi questa condizione nel choose per spegnere
- conditions:
    - condition: trigger
      id: illuminance_high
    - condition: numeric_state
      entity_id: sensor.presenza_camera_illuminance
      above: 50
  sequence:
    - target:
        entity_id: light.luce_salotto
      action: light.turn_off
```

---

Vuoi che ti crei la versione completa con entrambe le funzionalità?