csharp
using UnityEngine;
using UnityEngine.SceneManagement;
using CrazyGames;
public class CrazySDKInitializer : MonoBehaviour {
    void Start() {
        if (CrazySDK.IsAvailable) {
            CrazySDK.Init(() => { SceneManager.LoadScene("GameScene"); }); // SDK तैयार होने पर गेम लोड करें [1]
        } else { SceneManager.LoadScene("GameScene"); }
    }
}

EnemyAI.cs
using UnityEngine;
public class EnemyAI : MonoBehaviour {
    public int health = 100;
    public void TakeDamage(int amount) {
        health -= amount;
        if (health <= 0) Destroy(gameObject);
    }
}
SafePlayerPrefs.cs
using CrazyGames;
using UnityEngine;
public static class SafePlayerPrefs {
    static bool UseSDK => CrazySDK.IsAvailable && CrazySDK.IsInitialized;
    public static void SetInt(string key, int value) {
        if (UseSDK) CrazySDK.Data.SetInt(key, value); else PlayerPrefs.SetInt(key, value);
    }
    public static int GetInt(string key, int def = 0) {
        return UseSDK? CrazySDK.Data.GetInt(key, def) : PlayerPrefs.GetInt(key, def);
    }
}
AudioManager.cs
using UnityEngine;
public class AudioManager : MonoBehaviour {
    public static AudioManager instance;
    public AudioSource musicSource, sfxSource;
    public AudioClip backgroundMusic, gunShot, hitSound, footstep;
    void Awake() {
        if (instance == null) instance = this; else Destroy(gameObject);
        DontDestroyOnLoad(gameObject);
    }
    void Start() { musicSource.clip = backgroundMusic; musicSource.loop = true; musicSource.Play(); }
    public void PlayGun() { sfxSource.pitch = Random.Range(0.95f, 1.05f); sfxSource.PlayOneShot(gunShot); }
    public void PlayHit() => sfxSource.PlayOneShot(hitSound);
    public void PlayFootstep() => sfxSource.PlayOneShot(footstep);
}
Gun.cs
using UnityEngine;
public class Gun : MonoBehaviour {
    public Camera cam;
    public float range = 100f;
    public int damage = 50;
    void Update() {
        if (Input.GetMouseButtonDown(0)) Shoot(); // Mouse click for WebGL [2]
    }
    void Shoot() {
        AudioManager.instance.PlayGun();
        if (Physics.Raycast(cam.transform.position, cam.transform.forward, out RaycastHit hit, range)) {
            AudioManager.instance.PlayHit();
            EnemyAI enemy = hit.transform.GetComponent<EnemyAI>();
            if (enemy!= null) {
                enemy.TakeDamage(damage);
                ScoreManager.instance.AddScore(10);
                CoinManager.instance.AddCoins(5);
            }
        }
    }
}
Footsteps.cs
using UnityEngine;
public class Footsteps : MonoBehaviour {
    public CharacterController controller;
    public float stepDelay = 0.5f;
    float timer;
    void Update() {
        if (controller.velocity.magnitude > 0.1f) {
            timer += Time.deltaTime;
            if (timer >= stepDelay) { AudioManager.instance.PlayFootstep(); timer = 0f; }
        }
    }
}
EnemyAI.cs
using UnityEngine;
public class EnemyAI : MonoBehaviour {
    public int health = 100;
    public void TakeDamage(int amount) {
        health -= amount;
        if (health <= 0) Destroy(gameObject);
    }
}
Shop Manager
using UnityEngine;
public class ShopManager : MonoBehaviour {
    public void BuyWeapon(int price, string weaponKey) {
        if (CoinManager.instance.coins >= price) {
            CoinManager.instance.AddCoins(-price);
            SafePlayerPrefs.SetInt(weaponKey, 1);
        }
    }
}
Volume Control
using UnityEngine;
using UnityEngine.UI;
public class VolumeControl : MonoBehaviour {
    public Slider musicSlider, sfxSlider;
    void Start() {
        musicSlider.onValueChanged.AddListener(val => AudioManager.instance.musicSource.volume = val);
        sfxSlider.onValueChanged.AddListener(val => AudioManager.instance.sfxSource.volume = val);
    }
}
Ads Manager
using UnityEngine;
using CrazyGames;
public class AdsManager : MonoBehaviour {
    public static AdsManager instance;
    void Awake() => instance = this;
    public void ShowInterstitial() {
        CrazySDK.Ad.RequestAd(CrazyAdType.Midgame,
            () => { Time.timeScale = 0; AudioListener.pause = true; }, // Pause [3]
            (err) => { Time.timeScale = 1; AudioListener.pause = false; },
            () => { Time.timeScale = 1; AudioListener.pause = false; });
    }
    public void ShowRewarded() {
        CrazySDK.Ad.RequestAd(CrazyAdType.Rewarded,
            () => { Time.timeScale = 0; AudioListener.pause = true; },
            (err) => { Time.timeScale = 1; AudioListener.pause = false; },
            () => {
                Time.timeScale = 1; AudioListener.pause = false;
                CoinManager.instance.AddCoins(50); // Reward [3]
            });
    }
}
