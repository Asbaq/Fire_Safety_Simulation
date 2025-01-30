
# 🔥 Fire Safety Simulation

## 📌 Introduction
The **Fire Safety Simulation** is a VR-based training experience designed to educate users on fire safety procedures. The simulation includes interactive fire extinguishers, NPC reactions, emergency calls, movement mechanics, and realistic fire scenarios.

🔗 Video Trailer

https://youtu.be/Uwn3UwoQOwo

 ![image](https://github.com/user-attachments/assets/0d44f7bb-547d-4932-83e7-40fae4b9103e)


## 🔥 Features
- 🏃 **VR Hand & Player Movement**: XR-based hand interactions and continuous locomotion system.
- 🧯 **Fire Extinguisher Mechanism**: Simulates realistic pin pulling and fire suppression.
- 📞 **Emergency Call System**: Users can dial emergency numbers using a virtual phone.
- 🚪 **NPC AI Behavior**: NPCs react to fire alarms and move randomly in panic.
- 🔔 **Alarm System**: Triggers NPC reactions and scene changes.
- 🎵 **Immersive Audio**: Includes alarms, NPC screams, and extinguisher sounds.
- ⏳ **Simulation Status Tracking**: Tracks the number of fires extinguished and determines when the simulation is complete.

---

## 🏗️ How It Works

The simulation consists of various interactive objects and mechanics that contribute to an engaging fire safety training experience. Users can move using a VR controller, interact with objects, and follow safety protocols.

---

### 🖐 **Hand Interaction & VR Controls**
The **HandController** script manages user input from VR controllers for gripping and triggering interactions.

```csharp
using UnityEngine;
using UnityEngine.InputSystem;
using UnityEngine.XR.Interaction.Toolkit;

[RequireComponent(typeof(ActionBasedController))]
public class HandController : MonoBehaviour
{
    [SerializeField] InputActionReference PrimarySecondaryButtonsTouched;
    [SerializeField] Hand hand;
    [SerializeField] bool leftHand;
    ActionBasedController controller;

    private void Awake()
    {
        controller = GetComponent<ActionBasedController>();
    }

    void Update()
    {
        if(leftHand) hand.SetLeftGrip(controller.selectAction.action.ReadValue<float>());
        else hand.SetGrip(controller.selectAction.action.ReadValue<float>());
        hand.SetTrigger(controller.activateAction.action.ReadValue<float>());
        hand.SetThumb(PrimarySecondaryButtonsTouched.action.ReadValue<float>());
    }
}
```

---

### 🏃 **VR Locomotion (Continuous Movement)**
Handles player movement using a **ContinuousMoveProviderBase** system.

```csharp
using UnityEngine;
using UnityEngine.InputSystem;
using UnityEngine.XR.Interaction.Toolkit;

[RequireComponent(typeof(ContinuousMoveProviderBase))]
public class Movement : MonoBehaviour
{
    [SerializeField] InputActionReference runAction;
    [SerializeField] ContinuousMoveProviderBase moveSystem;
    [SerializeField] float runSpeed;
    [SerializeField] float walkSpeed;

    private void Awake()
    {
        if (!runAction) Debug.Log("Add the run action binding to the movement script in XR Origin!");
        if(!moveSystem) moveSystem = GetComponent<ContinuousMoveProviderBase>();
    }

    void Update()
    {
        if (!runAction) return;
        moveSystem.moveSpeed = runAction.action.ReadValue<float>() == 1 ? runSpeed : walkSpeed;
    }
}
```

---

### 📞 **Emergency Call System**
Users can enter a number on a virtual keypad and trigger an emergency call.

```csharp
using UnityEngine;
using TMPro;

public class PhoneInteraction : MonoBehaviour
{
    [SerializeField] TextMeshProUGUI numberToCall;
    [SerializeField] AudioClip emergencyCall, callDropped;
    [SerializeField] AudioClip[] dialing;
    [SerializeField] AudioSource source;

    private string numbersPressed = "";
    private int maxLengthNum = 6;
    private bool calling = false;

    private void OnTriggerEnter(Collider other)
    {
        string otherTag = other.gameObject.tag;
        int num;

        if (int.TryParse(otherTag, out num) && numbersPressed.Length < maxLengthNum)
        {
            numbersPressed += num;
            UpdateText();
            source.PlayOneShot(dialing[num]);
        }
        else if (otherTag == "Delete") EraseNums();
        else if (otherTag == "Call") Call();
    }

    private void Update()
    {
        if (calling && !source.isPlaying)
        {
            UpdateText();
            calling = false;
        }
    }

    private void UpdateText() => numberToCall.text = numbersPressed;

    private void EraseNums()
    {
        numbersPressed = "";
        numberToCall.text = numbersPressed;
    }

    private void Call()
    {
        if (!source || calling) return;
        
        if (numbersPressed == "112") StartCall(emergencyCall);
        else StartCall(callDropped);

        calling = true;
    }

    private void StartCall(AudioClip clipToPlay)
    {
        numberToCall.text = "Calling " + numbersPressed;
        source.PlayOneShot(clipToPlay);
    }
}
```

---

### 🤖 **NPC Behavior (Reaction to Fire)**
NPCs react to fire alarms by panicking and moving randomly.

```csharp
using UnityEngine;
using UnityEngine.AI;

public class NPCController : MonoBehaviour
{
    [SerializeField] float walkRadius;
    [SerializeField] float newDestinationTimer;

    NavMeshAgent agent;
    Vector3 finalDestination;
    bool findNewDest = true;

    void Start()
    {
        agent = GetComponent<NavMeshAgent>();
    }

    void Update()
    {
        if(findNewDest)
            StartCoroutine(RandomDestination());
    }

    IEnumerator RandomDestination()
    {
        Vector3 randomDirection = Random.insideUnitSphere * walkRadius + transform.position;
        NavMesh.SamplePosition(randomDirection, out NavMeshHit hit, walkRadius, 1);
        finalDestination = hit.position;

        agent.SetDestination(finalDestination);
        findNewDest = false;

        yield return new WaitForSeconds(newDestinationTimer);
        findNewDest = true;
    }
}
```

---

### 🧯 **Fire Extinguisher Mechanism**
Simulates pulling the pin and activating the extinguisher.

```csharp
using UnityEngine;

public class PinPull : MonoBehaviour
{
    [SerializeField] FireExtinguish extinguisherScript;
    [SerializeField] AudioSource audioSource;
    [SerializeField] GameObject particles;
    [SerializeField] Animator handlesAnim;
    Animator anim;

    void Start()
    {
        anim = GetComponent<Animator>();
    }

    public void PullPin()
    {
        anim.SetBool("pulledPin", true);
        extinguisherScript.isEnabled = true;
        audioSource.enabled = true;
        handlesAnim.enabled = true;
        particles.SetActive(true);
    }

    void OnTriggerEnter(Collider other)
    {
        if (other.CompareTag("Hand"))
            PullPin();
    }
}
```

---

### 🎮 **Simulation Status & Win Condition**
Tracks fires extinguished and determines when the simulation is complete.

```csharp
using UnityEngine;
using System;

public class SimulationStatus : MonoBehaviour
{
    [SerializeField] int amountFiresInSim;
    private int firesTurnedOff = 0;
    public static event EventHandler<float> GameWon;
    private bool won = false;
    float timer = 0;

    void Start()
    {
        Fire.fireOff += TurnFireOff;
    }

    void Update()
    {
        if (!won && WonGame())
        {
            GameWon?.Invoke(this, timer);
            won = true;
        }
        else if (!won)
        {
            timer += Time.deltaTime;
        }
    }

    void TurnFireOff() => firesTurnedOff++;

    bool WonGame() => firesTurnedOff == amountFiresInSim;

    private void OnDestroy()
    {
        Fire.fireOff -= TurnFireOff;
    }
}
```

---

## 🎯 Conclusion

The **Fire Safety Simulation** is an interactive and educational VR experience, designed to train users in emergency response situations. With realistic fire extinguishing mechanics, NPC reactions, and emergency call systems, the simulation provides a fully immersive training environment. 🚒🔥💨

