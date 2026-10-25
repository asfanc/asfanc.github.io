import { useCallback, useEffect, useRef, useState } from 'react';
import type { CSSProperties, PointerEvent as ReactPointerEvent } from 'react';
import {
  ArrowBigDown,
  ArrowBigUp,
  ArrowLeft,
  ArrowRight,
  ArrowUpRight,
  Check,
  ChevronLeft,
  ChevronRight,
  Coins,
  Gauge,
  LockKeyhole,
  Map as MapIcon,
  MoveHorizontal,
  Octagon,
  Pause,
  Play,
  RotateCcw,
  Trophy,
  Volume2,
  VolumeX,
  Wrench,
  Sparkles,
  X,
  Zap,
} from 'lucide-react';
import { CARS, EXHAUSTS, MAPS, MAX_MOD_LEVEL, RoadGame, emptyMods, exhaustPrice, neonCost, spoilerCost, upgradeCost } from './game';
import type { CarMods, CarSpec, GameMode, HudData, MapSpec, RunResult, StatKey, ThrottleMode } from './game';

interface Profile {
  credits: number;
  unlocked: string[];
  selected: string;
  map: string;
  throttle: ThrottleMode;
  upgrades: Record<string, CarMods>;
}

interface ScoreEntry {
  id: number;
  score: number;
  carId: string;
  mapId: string;
  passes: number;
  date: string;
}

type ModStat = StatKey;

const PROFILE_KEY = 'asaf-profile-v3';
const LEGACY_PROFILE_KEY = 'ters-serit-profile-v2';
const SCORES_KEY = 'ters-serit-scores-v1';
const INITIAL_HUD: HudData = { score: 0, earned: 0, passes: 0, combo: 0, kmh: 0, distance: 0, exhaust: 0 };

function sanitizeMods(value: unknown): CarMods {
  const mods = emptyMods();
  if (typeof value !== 'object' || value === null) return mods;
  const record = value as Record<string, unknown>;
  for (const key of ['manevra', 'hiz', 'fren'] as ModStat[]) {
    const level = record[key];
    if (typeof level === 'number' && Number.isFinite(level)) {
      mods[key] = Math.max(0, Math.min(MAX_MOD_LEVEL, Math.floor(level)));
    }
  }
  if (typeof record.exhaust === 'number' && Number.isFinite(record.exhaust)) {
    mods.exhaust = Math.max(0, Math.min(4, Math.floor(record.exhaust)));
  }
  if (typeof record.spoiler === 'number' && record.spoiler > 0) mods.spoiler = 1;
  if (typeof record.neon === 'number' && record.neon > 0) mods.neon = 1;
  if (Array.isArray(record.ownedExhaust)) {
    mods.ownedExhaust = Array.from(new Set(
      record.ownedExhaust.filter((id): id is number => typeof id === 'number' && id >= 0 && id <= 4),
    ));
  }
  if (!mods.ownedExhaust.includes(0)) mods.ownedExhaust.unshift(0);
  if (!mods.ownedExhaust.includes(mods.exhaust)) mods.ownedExhaust.push(mods.exhaust);
  return mods;
}

function loadProfile(): Profile {
  const fallback: Profile = {
    credits: 0,
    unlocked: [CARS[0].id],
    selected: CARS[0].id,
    map: MAPS[0].id,
    throttle: 'auto',
    upgrades: {},
  };
  try {
    const saved = JSON.parse(localStorage.getItem(PROFILE_KEY) || 'null') as Partial<Profile> | null;
    if (saved) {
      const unlocked = Array.isArray(saved.unlocked)
        ? saved.unlocked.filter((id): id is string => typeof id === 'string' && CARS.some((car) => car.id === id))
        : [];
      if (!unlocked.includes(CARS[0].id)) unlocked.unshift(CARS[0].id);
      const upgrades: Record<string, CarMods> = {};
      if (saved.upgrades && typeof saved.upgrades === 'object') {
        for (const [carId, mods] of Object.entries(saved.upgrades)) {
          if (CARS.some((car) => car.id === carId)) upgrades[carId] = sanitizeMods(mods);
        }
      }
      return {
        credits: Number.isFinite(saved.credits) ? Math.max(0, Math.floor(saved.credits as number)) : 0,
        unlocked,
        selected: typeof saved.selected === 'string' && unlocked.includes(saved.selected) ? saved.selected : CARS[0].id,
        map: typeof saved.map === 'string' && MAPS.some((map) => map.id === saved.map) ? saved.map : MAPS[0].id,
        throttle: saved.throttle === 'manual' ? 'manual' : 'auto',
        upgrades,
      };
    }
    // Migrate credits from the previous version so veterans keep their wallet.
    const legacy = JSON.parse(localStorage.getItem(LEGACY_PROFILE_KEY) || 'null') as { credits?: unknown } | null;
    if (legacy && Number.isFinite(legacy.credits)) {
      return { ...fallback, credits: Math.max(0, Math.floor(legacy.credits as number)) };
    }
  } catch {
    // Corrupt storage falls through to the fresh profile.
  }
  return fallback;
}

function loadScores(): ScoreEntry[] {
  try {
    const saved = JSON.parse(localStorage.getItem(SCORES_KEY) || '[]') as ScoreEntry[];
    if (!Array.isArray(saved)) return [];
    return saved
      .filter((entry) => Number.isFinite(entry.score) && CARS.some((car) => car.id === entry.carId))
      .map((entry) => ({ ...entry, mapId: typeof entry.mapId === 'string' ? entry.mapId : MAPS[0].id }))
      .sort((a, b) => b.score - a.score)
      .slice(0, 5);
  } catch {
    return [];
  }
}

const scoreText = (score: number) => Math.floor(score).toString().padStart(6, '0');
const creditText = (credits: number) => Math.floor(credits).toLocaleString('tr-TR');
const distanceText = (meters: number) => (meters >= 1000 ? `${(meters / 1000).toFixed(2)} KM` : `${Math.floor(meters)} M`);

function CarPreview({ car }: { car: CarSpec }) {
  const gradientId = `car-paint-${car.id}`;
  const glowId = `car-glow-${car.id}`;
  const paths = [
    'M40 12 Q60 4 80 12 L94 48 L96 162 L84 207 Q60 218 36 207 L24 162 L26 48 Z',
    'M43 7 Q60 2 77 7 L91 43 L98 169 L82 211 Q60 219 38 211 L22 169 L29 43 Z',
    'M34 12 L86 12 L99 46 L100 174 L88 211 L32 211 L20 174 L21 46 Z',
    'M48 5 L72 5 L91 38 L104 172 L84 214 L36 214 L16 172 L29 38 Z',
  ];

  return (
    <svg className="preview-svg" viewBox="0 0 120 220" fill="none" role="img" aria-label={`${car.brand} ${car.name} aracının üstten görünümü`}>
      <defs>
        <linearGradient id={gradientId} x1="8" y1="0" x2="106" y2="0" gradientUnits="userSpaceOnUse">
          <stop stopColor="#17292c" />
          <stop offset="0.2" stopColor={car.color} />
          <stop offset="0.51" stopColor={car.color} />
          <stop offset="0.84" stopColor={car.color} />
          <stop offset="1" stopColor="#17292c" />
        </linearGradient>
        <filter id={glowId} x="-70%" y="-50%" width="240%" height="200%">
          <feGaussianBlur stdDeviation="13" />
        </filter>
      </defs>
      <ellipse cx="60" cy="122" rx="49" ry="98" fill={car.color} opacity="0.28" filter={`url(#${glowId})`} />
      <path d="M16 43 H29 V74 H16 Z M91 43 H104 V74 H91 Z M16 147 H29 V184 H16 Z M91 147 H104 V184 H91 Z" fill="#050a0d" />
      <path d="M19 50 H23 V65 H19 Z M97 50 H101 V65 H97 Z M19 156 H23 V174 H19 Z M97 156 H101 V174 H97 Z" fill="#546165" />
      <path d={paths[car.kind]} fill={`url(#${gradientId})`} stroke="rgba(255,255,255,.75)" strokeWidth="1.7" />
      <path d="M38 50 H82 L86 73 H34 Z" fill="#182f35" opacity="0.63" />
      <path d="M38 82 H82 L88 139 H32 Z" fill="#0c252e" stroke="#c4e6df" strokeWidth="1.6" />
      <path d="M39 86 H81 L85 108 H35 Z" fill="#7cbfcb" opacity="0.61" />
      <path d="M35 142 H85 L80 175 H40 Z" fill="#102229" />
      <path d="M25 101 H31 V137 H25 Z M89 101 H95 V137 H89 Z" fill="#0c1c20" opacity="0.45" />
      {car.kind === 2 ? (
        <path d="M47 17 H53 V57 H47 Z M67 17 H73 V57 H67 Z" fill="#fcf8e9" opacity="0.7" />
      ) : car.kind === 3 ? (
        <path d="M60 15 L68 58 L60 50 L52 58 Z" fill="#f4f4ed" opacity="0.72" />
      ) : (
        <path d="M58 13 H62 V57 H58 Z" fill="#f5f5df" opacity="0.72" />
      )}
      <path d="M32 18 H49 V24 H30 Z M71 18 H88 L90 24 H71 Z" fill="#fff9da" />
      <path d="M29 194 H47 V200 H31 Z M73 194 H91 L89 200 H73 Z" fill="#ff6054" />
      <path d="M43 183 H77" stroke="#081c21" strokeWidth="2" />
    </svg>
  );
}

function ThrottleToggle({ value, onChange, compact = false }: { value: ThrottleMode; onChange: (mode: ThrottleMode) => void; compact?: boolean }) {
  return (
    <div className={`throttle-toggle ${compact ? 'throttle-compact' : ''}`} role="group" aria-label="Hızlanma modu">
      <button className={value === 'auto' ? 'throttle-active' : ''} onClick={() => onChange('auto')} title="Otomatik hız: gaz ve fren takviye için">
        <Gauge size={15} /> OTO GAZ
      </button>
      <button className={value === 'manual' ? 'throttle-active' : ''} onClick={() => onChange('manual')} title="Manuel: hız tamamen gaz ve fren pedallarında">
        <Zap size={15} /> MANUEL
      </button>
    </div>
  );
}

function MapRow({ maps, selectedId, onSelect, mini = false }: { maps: MapSpec[]; selectedId: string; onSelect: (id: string) => void; mini?: boolean }) {
  return (
    <div className={`map-row ${mini ? 'map-row-mini' : ''}`}>
      {maps.map((map) => (
        <button
          key={map.id}
          className={`map-card ${selectedId === map.id ? 'map-card-active' : ''}`}
          onClick={() => onSelect(map.id)}
          title={map.description}
        >
          <i style={{ background: map.color }} />
          <span className="map-card-name">{map.short}</span>
          <span className="map-card-mult">x{map.multiplier.toFixed(2)}</span>
        </button>
      ))}
    </div>
  );
}

const MOD_META: Array<{ stat: ModStat; label: string; hint: string; icon: React.ReactNode }> = [
  { stat: 'manevra', label: 'MANEVRA', hint: 'Daha keskin şerit değiştirme', icon: <MoveHorizontal size={16} /> },
  { stat: 'hiz', label: 'HIZ', hint: 'Daha yüksek hız ve kazanç', icon: <Zap size={16} /> },
  { stat: 'fren', label: 'FREN', hint: 'Daha kısa fren mesafesi', icon: <Octagon size={16} /> },
];

type SoundType = 'start' | 'near' | 'tight' | 'crash' | 'buy' | 'click' | 'level' | 'pop';

export default function App() {
  const canvasRef = useRef<HTMLCanvasElement>(null);
  const gameRef = useRef<RoadGame | null>(null);
  const audioRef = useRef<AudioContext | null>(null);
  const engineRef = useRef<{ osc: OscillatorNode; osc2: OscillatorNode; gain: GainNode; filter: BiquadFilterNode } | null>(null);
  const screenRef = useRef<GameMode>('menu');
  const returnScreenRef = useRef<GameMode>('menu');
  const [screen, setScreen] = useState<GameMode>('menu');
  const [profile, setProfile] = useState<Profile>(loadProfile);
  const profileRef = useRef(profile);
  const [scores, setScores] = useState<ScoreEntry[]>(loadScores);
  const scoresRef = useRef(scores);
  const [hud, setHud] = useState<HudData>(INITIAL_HUD);
  const [result, setResult] = useState<RunResult | null>(null);
  const [newRecord, setNewRecord] = useState(false);
  const [lastScoreId, setLastScoreId] = useState(0);
  const [garageFocusId, setGarageFocusId] = useState(profile.selected);
  const [soundOn, setSoundOn] = useState(true);
  const soundRef = useRef(true);

  const changeScreen = useCallback((next: GameMode) => {
    screenRef.current = next;
    setScreen(next);
    gameRef.current?.setMode(next);
  }, []);

  const playSound = useCallback((type: SoundType) => {
    if (!soundRef.current) return;
    try {
      const audio = audioRef.current ?? new AudioContext();
      audioRef.current = audio;
      if (audio.state === 'suspended') void audio.resume();
      const now = audio.currentTime;
      const tone = (from: number, to: number, duration: number, delay: number, volume: number, wave: OscillatorType = 'triangle') => {
        const oscillator = audio.createOscillator();
        const gain = audio.createGain();
        oscillator.type = wave;
        oscillator.frequency.setValueAtTime(from, now + delay);
        oscillator.frequency.exponentialRampToValueAtTime(to, now + delay + duration);
        gain.gain.setValueAtTime(0.001, now + delay);
        gain.gain.exponentialRampToValueAtTime(volume, now + delay + 0.015);
        gain.gain.exponentialRampToValueAtTime(0.001, now + delay + duration);
        oscillator.connect(gain).connect(audio.destination);
        oscillator.start(now + delay);
        oscillator.stop(now + delay + duration + 0.02);
      };

      if (type === 'start') {
        tone(165, 330, 0.2, 0, 0.075, 'sawtooth');
        tone(330, 660, 0.24, 0.13, 0.065, 'triangle');
      } else if (type === 'near' || type === 'tight') {
        tone(type === 'tight' ? 510 : 400, type === 'tight' ? 1040 : 760, 0.13, 0, 0.07);
        tone(type === 'tight' ? 770 : 600, type === 'tight' ? 1440 : 960, 0.16, 0.09, 0.06);
      } else if (type === 'buy') {
        tone(320, 640, 0.17, 0, 0.065);
        tone(480, 960, 0.2, 0.12, 0.065);
        tone(720, 1440, 0.24, 0.23, 0.055);
      } else if (type === 'level') {
        tone(392, 392, 0.12, 0, 0.07, 'square');
        tone(523, 523, 0.12, 0.13, 0.07, 'square');
        tone(659, 659, 0.2, 0.26, 0.07, 'square');
      } else if (type === 'pop') {
        tone(180, 90, 0.08, 0, 0.12, 'sawtooth');
        tone(520, 140, 0.12, 0.04, 0.08, 'square');
      } else if (type === 'crash') {
        tone(180, 38, 0.42, 0, 0.16, 'sawtooth');
        const buffer = audio.createBuffer(1, Math.floor(audio.sampleRate * 0.3), audio.sampleRate);
        const samples = buffer.getChannelData(0);
        for (let i = 0; i < samples.length; i += 1) samples[i] = Math.random() * 2 - 1;
        const noise = audio.createBufferSource();
        const filter = audio.createBiquadFilter();
        const gain = audio.createGain();
        noise.buffer = buffer;
        filter.type = 'lowpass';
        filter.frequency.setValueAtTime(2600, now);
        filter.frequency.exponentialRampToValueAtTime(170, now + 0.28);
        gain.gain.setValueAtTime(0.16, now);
        gain.gain.exponentialRampToValueAtTime(0.001, now + 0.3);
        noise.connect(filter).connect(gain).connect(audio.destination);
        noise.start(now);
      } else {
        tone(490, 650, 0.07, 0, 0.035);
      }
    } catch {
      // The game remains playable if a browser disallows Web Audio.
    }
  }, []);

  const stopEngine = useCallback(() => {
    const engine = engineRef.current;
    engineRef.current = null;
    if (!engine) return;
    try {
      const audio = audioRef.current;
      if (audio) engine.gain.gain.setTargetAtTime(0.0001, audio.currentTime, 0.06);
      window.setTimeout(() => {
        try { engine.osc.stop(); engine.osc2.stop(); } catch { /* Already stopped. */ }
      }, 250);
    } catch {
      // Engine sound is optional decoration.
    }
  }, []);

  const startEngine = useCallback(() => {
    if (!soundRef.current || engineRef.current) return;
    try {
      const audio = audioRef.current ?? new AudioContext();
      audioRef.current = audio;
      if (audio.state === 'suspended') void audio.resume();
      const osc = audio.createOscillator();
      osc.type = 'sawtooth';
      osc.frequency.value = 70;
      const osc2 = audio.createOscillator();
      osc2.type = 'square';
      osc2.frequency.value = 35;
      const filter = audio.createBiquadFilter();
      filter.type = 'lowpass';
      filter.frequency.value = 420;
      const gain = audio.createGain();
      gain.gain.value = 0.0001;
      gain.gain.setTargetAtTime(0.022, audio.currentTime, 0.3);
      osc.connect(filter);
      osc2.connect(filter);
      filter.connect(gain);
      gain.connect(audio.destination);
      osc.start();
      osc2.start();
      engineRef.current = { osc, osc2, gain, filter };
    } catch {
      // The game remains playable if a browser disallows Web Audio.
    }
  }, []);

  useEffect(() => {
    if (!canvasRef.current) return;
    const game = new RoadGame(canvasRef.current, {
      onHud: setHud,
      onNear: (amount, tight) => {
        setProfile((current) => {
          const next = { ...current, credits: current.credits + amount };
          profileRef.current = next;
          return next;
        });
        playSound(tight ? 'tight' : 'near');
        navigator.vibrate?.(tight ? 38 : 18);
      },
      onImpact: () => {
        playSound('crash');
        navigator.vibrate?.([80, 35, 100]);
      },
      onLevel: () => {
        playSound('level');
        navigator.vibrate?.(30);
      },
      onPop: () => {
        playSound('pop');
        navigator.vibrate?.(12);
      },
      onGameOver: (run) => {
        setResult(run);
        setNewRecord(run.score > (scoresRef.current[0]?.score ?? 0));
        const entry: ScoreEntry = {
          id: Date.now(),
          score: run.score,
          carId: run.carId,
          mapId: run.mapId,
          passes: run.passes,
          date: new Date().toLocaleDateString('tr-TR', { day: '2-digit', month: '2-digit', year: '2-digit' }),
        };
        setLastScoreId(entry.id);
        setScores((current) => {
          const next = [...current, entry].sort((a, b) => b.score - a.score).slice(0, 5);
          scoresRef.current = next;
          return next;
        });
        changeScreen('gameover');
      },
    });
    gameRef.current = game;
    game.setCar(profileRef.current.selected);
    game.setMods(profileRef.current.upgrades[profileRef.current.selected] ?? emptyMods());
    game.setMap(profileRef.current.map);
    game.setThrottleMode(profileRef.current.throttle);
    return () => {
      game.dispose();
      gameRef.current = null;
    };
  }, [changeScreen, playSound]);

  useEffect(() => {
    profileRef.current = profile;
    const game = gameRef.current;
    if (game) {
      game.setCar(profile.selected);
      game.setMods(profile.upgrades[profile.selected] ?? emptyMods());
      game.setMap(profile.map);
      game.setThrottleMode(profile.throttle);
    }
    try { localStorage.setItem(PROFILE_KEY, JSON.stringify(profile)); } catch { /* Storage is optional. */ }
  }, [profile]);

  useEffect(() => {
    scoresRef.current = scores;
    try { localStorage.setItem(SCORES_KEY, JSON.stringify(scores)); } catch { /* Storage is optional. */ }
  }, [scores]);

  useEffect(() => {
    if (screen === 'playing' && soundOn) startEngine();
    else stopEngine();
  }, [screen, soundOn, startEngine, stopEngine]);

  useEffect(() => {
    const engine = engineRef.current;
    const audio = audioRef.current;
    if (!engine || !audio || screen !== 'playing') return;
    const now = audio.currentTime;
    const ex = Math.max(0, Math.min(4, hud.exhaust | 0));
    const mul = [0.32, 0.48, 0.55, 0.22, 0.78][ex];
    const filt = [260, 420, 980, 220, 1500][ex];
    const vol = [0.014, 0.022, 0.024, 0.03, 0.026][ex];
    const freq = 42 + hud.kmh * mul;
    engine.osc.frequency.setTargetAtTime(freq, now, 0.08);
    engine.osc2.frequency.setTargetAtTime(ex === 2 ? freq * 2.4 : freq / 2, now, 0.08);
    engine.filter.frequency.setTargetAtTime(filt + hud.kmh * (1.4 + ex * 0.4), now, 0.1);
    engine.gain.gain.setTargetAtTime(vol, now, 0.12);
  }, [hud.exhaust, hud.kmh, screen]);

  useEffect(() => () => {
    stopEngine();
    void audioRef.current?.close();
  }, [stopEngine]);

  const startRun = useCallback((carId?: string) => {
    const chosenId = carId ?? profileRef.current.selected;
    if (chosenId !== profileRef.current.selected) {
      const next = { ...profileRef.current, selected: chosenId };
      profileRef.current = next;
      setProfile(next);
    }
    setResult(null);
    setNewRecord(false);
    setHud(INITIAL_HUD);
    gameRef.current?.setCar(chosenId);
    gameRef.current?.setMods(profileRef.current.upgrades[chosenId] ?? emptyMods());
    gameRef.current?.start();
    changeScreen('playing');
    playSound('start');
  }, [changeScreen, playSound]);

  const pauseGame = useCallback(() => {
    if (screenRef.current === 'playing') {
      changeScreen('paused');
      playSound('click');
    }
  }, [changeScreen, playSound]);

  const resumeGame = useCallback(() => {
    if (screenRef.current === 'paused') {
      changeScreen('playing');
      playSound('click');
    }
  }, [changeScreen, playSound]);

  const closeOverlay = useCallback(() => {
    changeScreen(returnScreenRef.current);
    playSound('click');
  }, [changeScreen, playSound]);

  const openGarage = () => {
    returnScreenRef.current = screenRef.current;
    setGarageFocusId(profileRef.current.selected);
    changeScreen('garage');
    playSound('click');
  };

  const openScores = () => {
    returnScreenRef.current = screenRef.current;
    changeScreen('scores');
    playSound('click');
  };

  const buyCar = (car: CarSpec) => {
    const current = profileRef.current;
    if (current.unlocked.includes(car.id) || current.credits < car.price) return;
    const next = {
      ...current,
      credits: current.credits - car.price,
      unlocked: [...current.unlocked, car.id],
      selected: car.id,
    };
    profileRef.current = next;
    setProfile(next);
    playSound('buy');
  };

  const buyUpgrade = (car: CarSpec, stat: ModStat) => {
    const current = profileRef.current;
    if (!current.unlocked.includes(car.id)) return;
    const mods = current.upgrades[car.id] ?? emptyMods();
    const level = mods[stat];
    if (level >= MAX_MOD_LEVEL) return;
    const cost = upgradeCost(car, level);
    if (current.credits < cost) return;
    const next = {
      ...current,
      credits: current.credits - cost,
      upgrades: { ...current.upgrades, [car.id]: { ...mods, [stat]: level + 1 } },
    };
    profileRef.current = next;
    setProfile(next);
    playSound('buy');
  };

  const buyExhaust = (car: CarSpec, exhaustId: number) => {
    const current = profileRef.current;
    if (!current.unlocked.includes(car.id)) return;
    const spec = EXHAUSTS[exhaustId];
    if (!spec) return;
    const mods = current.upgrades[car.id] ?? emptyMods();
    if (mods.ownedExhaust.includes(exhaustId)) {
      const next = { ...current, upgrades: { ...current.upgrades, [car.id]: { ...mods, exhaust: exhaustId } } };
      profileRef.current = next;
      setProfile(next);
      playSound('click');
      return;
    }
    const cost = exhaustPrice(car, spec);
    if (current.credits < cost) return;
    const next = {
      ...current,
      credits: current.credits - cost,
      upgrades: { ...current.upgrades, [car.id]: { ...mods, exhaust: exhaustId, ownedExhaust: [...mods.ownedExhaust, exhaustId] } },
    };
    profileRef.current = next;
    setProfile(next);
    playSound('buy');
  };

  const buyKit = (car: CarSpec, kind: 'spoiler' | 'neon') => {
    const current = profileRef.current;
    if (!current.unlocked.includes(car.id)) return;
    const mods = current.upgrades[car.id] ?? emptyMods();
    if (mods[kind] > 0) return;
    const cost = kind === 'spoiler' ? spoilerCost(car) : neonCost(car);
    if (current.credits < cost) return;
    const next = {
      ...current,
      credits: current.credits - cost,
      upgrades: { ...current.upgrades, [car.id]: { ...mods, [kind]: 1 } },
    };
    profileRef.current = next;
    setProfile(next);
    playSound('buy');
  };

  const selectMap = (mapId: string) => {
    if (profileRef.current.map === mapId) return;
    const next = { ...profileRef.current, map: mapId };
    profileRef.current = next;
    setProfile(next);
    playSound('click');
  };

  const selectThrottle = (mode: ThrottleMode) => {
    if (profileRef.current.throttle === mode) return;
    const next = { ...profileRef.current, throttle: mode };
    profileRef.current = next;
    setProfile(next);
    playSound('click');
  };

  const toggleSound = () => {
    const next = !soundRef.current;
    soundRef.current = next;
    setSoundOn(next);
    if (next) playSound('click');
  };

  useEffect(() => {
    const onKeyDown = (event: KeyboardEvent) => {
      const key = event.key.toLowerCase();
      const currentScreen = screenRef.current;
      if ((key === 'enter' || key === ' ') && event.target instanceof HTMLElement && event.target.closest('button')) return;
      if (key === 'escape' || ((currentScreen === 'playing' || currentScreen === 'paused') && ['arrowleft', 'arrowright', 'arrowup', 'arrowdown', ' '].includes(key)) || ((currentScreen === 'menu' || currentScreen === 'gameover') && key === ' ')) event.preventDefault();
      if (currentScreen === 'playing') {
        if (key === 'a' || key === 'arrowleft') gameRef.current?.setDirection('left', true);
        if (key === 'd' || key === 'arrowright') gameRef.current?.setDirection('right', true);
        if (key === 'w' || key === 'arrowup') gameRef.current?.setPedal('gas', true);
        if (key === 's' || key === 'arrowdown') gameRef.current?.setPedal('brake', true);
        if ((key === 'p' || key === 'escape' || key === ' ') && !event.repeat) pauseGame();
      } else if (currentScreen === 'paused') {
        if (['p', 'escape', 'enter', ' '].includes(key) && !event.repeat) resumeGame();
        if (key === 'r' && !event.repeat) startRun();
      } else if (currentScreen === 'menu' || currentScreen === 'gameover') {
        if ((key === 'enter' || key === ' ' || (currentScreen === 'gameover' && key === 'r')) && !event.repeat) startRun();
      } else if (key === 'escape') {
        closeOverlay();
      }
    };
    const onKeyUp = (event: KeyboardEvent) => {
      const key = event.key.toLowerCase();
      if (key === 'a' || key === 'arrowleft') gameRef.current?.setDirection('left', false);
      if (key === 'd' || key === 'arrowright') gameRef.current?.setDirection('right', false);
      if (key === 'w' || key === 'arrowup') gameRef.current?.setPedal('gas', false);
      if (key === 's' || key === 'arrowdown') gameRef.current?.setPedal('brake', false);
    };
    const onBlur = () => {
      gameRef.current?.setDirection('left', false);
      gameRef.current?.setDirection('right', false);
      gameRef.current?.setPedal('gas', false);
      gameRef.current?.setPedal('brake', false);
      if (screenRef.current === 'playing') pauseGame();
    };
    const onVisibility = () => { if (document.hidden) onBlur(); };
    window.addEventListener('keydown', onKeyDown);
    window.addEventListener('keyup', onKeyUp);
    window.addEventListener('blur', onBlur);
    document.addEventListener('visibilitychange', onVisibility);
    return () => {
      window.removeEventListener('keydown', onKeyDown);
      window.removeEventListener('keyup', onKeyUp);
      window.removeEventListener('blur', onBlur);
      document.removeEventListener('visibilitychange', onVisibility);
    };
  }, [closeOverlay, pauseGame, resumeGame, startRun]);

  const holdDown = (action: 'left' | 'right' | 'gas' | 'brake', event: ReactPointerEvent<HTMLButtonElement>) => {
    event.preventDefault();
    event.stopPropagation();
    event.currentTarget.setPointerCapture(event.pointerId);
    if (action === 'left' || action === 'right') gameRef.current?.setDirection(action, true);
    else gameRef.current?.setPedal(action, true);
  };

  const holdUp = (action: 'left' | 'right' | 'gas' | 'brake', event: ReactPointerEvent<HTMLButtonElement>) => {
    event.preventDefault();
    event.stopPropagation();
    if (action === 'left' || action === 'right') gameRef.current?.setDirection(action, false);
    else gameRef.current?.setPedal(action, false);
  };

  const selectedCar = CARS.find((car) => car.id === profile.selected) ?? CARS[0];
  const selectedMap = MAPS.find((map) => map.id === profile.map) ?? MAPS[0];
  const focusedCar = CARS.find((car) => car.id === garageFocusId) ?? CARS[0];
  const focusedMods = profile.upgrades[focusedCar.id] ?? emptyMods();
  const focusedOwned = profile.unlocked.includes(focusedCar.id);
  const canAfford = profile.credits >= focusedCar.price;
  const bestScore = scores[0]?.score ?? 0;
  const resultCar = CARS.find((car) => car.id === (result?.carId ?? '')) ?? selectedCar;
  const resultMap = MAPS.find((map) => map.id === (result?.mapId ?? '')) ?? selectedMap;

  const controlValue = focusedCar.handling * (1 + focusedMods.manevra * 0.09);
  const speedValue = Math.round(focusedCar.topSpeed * (1 + focusedMods.hiz * 0.04) * 300);
  const brakeValue = Math.round(focusedCar.brake * 100 + focusedMods.fren * 12);

  return (
    <div className={`app-shell screen-${screen}`} style={{ '--car-accent': selectedCar.color } as CSSProperties}>
      <canvas
        ref={canvasRef}
        className="game-canvas"
        aria-label="Ters Şerit oyun alanı"
        onPointerDown={(event) => {
          if (gameRef.current?.pointerDown(event.clientX)) event.currentTarget.setPointerCapture(event.pointerId);
        }}
        onPointerMove={(event) => gameRef.current?.pointerMove(event.clientX)}
        onPointerUp={() => gameRef.current?.pointerUp()}
        onPointerCancel={() => gameRef.current?.pointerUp()}
        onLostPointerCapture={() => gameRef.current?.pointerUp()}
      />
      <div className="world-shade" aria-hidden="true" />
      <div className="world-grain" aria-hidden="true" />

      {screen === 'menu' && (
        <div className="menu-layer">
          <header className="site-header">
            <div className="brand-lockup" aria-label="ASAF">
              <span className="brand-symbol"><span /><span /></span>
              <span className="brand-type">ASAF</span>
              <span className="brand-edition">034 GARAGE</span>
            </div>
            <div className="header-actions">
              <div className="header-wallet" title="Toplam kredi">
                <Coins size={17} strokeWidth={2.1} />
                <strong>{creditText(profile.credits)}</strong>
                <span>KREDİ</span>
              </div>
              <button className="header-link" onClick={openGarage}>GARAJ <ArrowUpRight size={16} /></button>
              <button className="header-icon" onClick={openScores} aria-label="Rekorları aç" title="Rekorlar"><Trophy size={19} /></button>
              <button className="header-icon sound-button" onClick={toggleSound} aria-label={soundOn ? 'Sesi kapat' : 'Sesi aç'} title={soundOn ? 'Sesi kapat' : 'Sesi aç'}>
                {soundOn ? <Volume2 size={19} /> : <VolumeX size={19} />}
              </button>
            </div>
          </header>

          <main className="intro">
            <div className="intro-kicker"><span className="kicker-line" /> ASAF GARAGE / 15 ARAÇ</div>
            <h1><span>ASAF</span><span className="title-second">KAÇIŞ<span className="title-stop">.</span></span></h1>
            <p>Karşıdan gelen trafikte boşluğu bul. Kıl payı geç, kredi kazan, Tofaş’tan Bugatti’ye kadar 15 arabayı modifiye et.</p>
            <div className="intro-actions">
              <button className="primary-button" onClick={() => startRun()}>
                <span>YOLA ÇIK</span><span className="button-icon"><ArrowUpRight size={25} strokeWidth={2.4} /></span>
              </button>
              <button className="text-button" onClick={openGarage}>ARAÇLARI GÖR <ArrowRight size={18} /></button>
            </div>
            <div className="map-section">
              <span className="map-label"><MapIcon size={14} /> HARİTA SEÇ <b>x{selectedMap.multiplier.toFixed(2)} KAZANÇ</b></span>
              <MapRow maps={MAPS} selectedId={profile.map} onSelect={selectMap} />
              <p className="map-desc">{selectedMap.description}</p>
            </div>
            <div className="menu-settings">
              <ThrottleToggle value={profile.throttle} onChange={selectThrottle} />
              <button className="car-chip" onClick={openGarage} title="Garajı aç">
                <i style={{ background: selectedCar.color }} />
                <span>{selectedCar.brand} {selectedCar.name}</span>
                <ArrowUpRight size={15} />
              </button>
            </div>
            <div className="intro-controls">
              <span className="desktop-control-copy"><kbd>A</kbd><kbd>D</kbd><span>sür</span><kbd>W</kbd><span>gaz</span><kbd>S</kbd><span>fren</span></span>
              <span className="mobile-control-copy">Yön tuşları + gaz ve fren pedallarını kullan.</span>
            </div>
          </main>
        </div>
      )}

      {(screen === 'playing' || screen === 'paused') && (
        <div className={`game-ui ${screen === 'paused' ? 'game-ui-paused' : ''}`}>
          <div className="hud-top">
            <div className="hud-score">
              <span className="hud-label">PUAN <span className="hud-label-line" /></span>
              <strong>{scoreText(hud.score)}</strong>
              <span className="hud-best">EN İYİ <b>{scoreText(bestScore)}</b></span>
            </div>
            <div className="hud-speed">
              <strong>{hud.kmh}</strong>
              <span>KM/H</span>
              <em>{selectedMap.name}</em>
            </div>
            <div className="hud-right">
              <div className="hud-earned"><span>KAZANILAN</span><strong><span>+</span>{creditText(hud.earned)}</strong><small>KREDİ</small></div>
              <button className="pause-button" onClick={pauseGame} aria-label="Oyunu duraklat" title="Duraklat (P)"><Pause size={22} fill="currentColor" strokeWidth={1.5} /></button>
            </div>
          </div>
          {hud.combo > 1 && <div className="combo-readout" key={hud.combo}><strong>x{hud.combo}</strong><span>SERİ<br />GEÇİŞ</span></div>}
          <div className="hud-bottom">
            <span>A / D SÜR &nbsp;•&nbsp; W GAZ &nbsp;•&nbsp; S FREN</span>
            <span>{String(hud.passes).padStart(2, '0')} GEÇİŞ &nbsp;•&nbsp; {distanceText(hud.distance)}</span>
          </div>
          <div className="touch-controls" aria-label="Dokunmatik sürüş kontrolleri">
            <div className="pedal-group steer-group">
              <button aria-label="Sola sür" onPointerDown={(event) => holdDown('left', event)} onPointerUp={(event) => holdUp('left', event)} onPointerCancel={(event) => holdUp('left', event)} onLostPointerCapture={(event) => holdUp('left', event)} onContextMenu={(event) => event.preventDefault()}><ChevronLeft size={39} strokeWidth={2.3} /></button>
              <button aria-label="Sağa sür" onPointerDown={(event) => holdDown('right', event)} onPointerUp={(event) => holdUp('right', event)} onPointerCancel={(event) => holdUp('right', event)} onLostPointerCapture={(event) => holdUp('right', event)} onContextMenu={(event) => event.preventDefault()}><ChevronRight size={39} strokeWidth={2.3} /></button>
            </div>
            <div className="pedal-group pedal-group-right">
              <button className="pedal-brake" aria-label="Fren" onPointerDown={(event) => holdDown('brake', event)} onPointerUp={(event) => holdUp('brake', event)} onPointerCancel={(event) => holdUp('brake', event)} onLostPointerCapture={(event) => holdUp('brake', event)} onContextMenu={(event) => event.preventDefault()}><ArrowBigDown size={34} strokeWidth={2} /><span>FREN</span></button>
              <button className="pedal-gas" aria-label="Gaz" onPointerDown={(event) => holdDown('gas', event)} onPointerUp={(event) => holdUp('gas', event)} onPointerCancel={(event) => holdUp('gas', event)} onLostPointerCapture={(event) => holdUp('gas', event)} onContextMenu={(event) => event.preventDefault()}><ArrowBigUp size={34} strokeWidth={2} /><span>GAZ</span></button>
            </div>
          </div>
        </div>
      )}

      {screen === 'paused' && (
        <div className="state-overlay pause-overlay">
          <div className="state-content pause-content">
            <span className="section-kicker"><span /> KISA BİR MOLA</span>
            <h2>MOLA<br /><em>VERDİN.</em></h2>
            <p>Yol seni bekliyor. Hazır olunca devam et.</p>
            <div className="pause-settings">
              <ThrottleToggle value={profile.throttle} onChange={selectThrottle} compact />
              <MapRow maps={MAPS} selectedId={profile.map} onSelect={selectMap} mini />
            </div>
            <div className="state-actions">
              <button className="primary-button" onClick={resumeGame}><span>DEVAM ET</span><span className="button-icon"><Play size={21} fill="currentColor" /></span></button>
              <button className="text-button" onClick={() => startRun()}>YENİDEN BAŞLA <RotateCcw size={17} /></button>
            </div>
            <button className="quiet-link" onClick={() => changeScreen('menu')}>ANA MENÜYE DÖN</button>
          </div>
        </div>
      )}

      {screen === 'gameover' && (
        <div className="state-overlay end-overlay">
          <div className="end-topbar"><div className="small-wordmark">ASAF <span>/</span> 034</div><button onClick={() => changeScreen('menu')}>ANA MENÜ <X size={17} /></button></div>
          <div className="end-layout">
            <div className="end-main">
              <span className="section-kicker"><span /> {newRecord ? 'YENİ REKOR!' : 'SÜRÜŞ SONA ERDİ'}</span>
              <h2>YOL<br /><em>BİTTİ.</em></h2>
              <div className="end-score"><span>BU SÜRÜŞÜN PUANI</span><strong>{scoreText(result?.score ?? hud.score)}</strong></div>
              <div className="end-run-meta">{resultCar.brand} {resultCar.name} • {resultMap.name}</div>
              <div className="end-stats-grid">
                <span><b>+{creditText(result?.earned ?? hud.earned)}</b> KREDİ</span>
                <span><b>{String(result?.passes ?? hud.passes).padStart(2, '0')}</b> GEÇİŞ</span>
                <span><b>{distanceText(result?.distance ?? hud.distance)}</b> MESAFE</span>
                <span><b>{result?.topKmh ?? hud.kmh}</b> KM/H MAX</span>
              </div>
              <div className="end-actions">
                <button className="primary-button" onClick={() => startRun()}><span>TEKRAR DENE</span><span className="button-icon"><RotateCcw size={23} /></span></button>
                <button className="text-button" onClick={openGarage}>GARAJA GİT <ArrowRight size={18} /></button>
              </div>
              <span className="restart-hint">ENTER VEYA R İLE ANINDA YENİDEN BAŞLA</span>
            </div>
            <div className="end-leaders">
              <div className="leader-heading"><Trophy size={18} /><span>EN İYİ SÜRÜŞLER</span></div>
              {scores.slice(0, 3).map((entry, index) => (
                <div className={`leader-mini-row ${entry.id === lastScoreId ? 'leader-current' : ''}`} key={entry.id}>
                  <span>{String(index + 1).padStart(2, '0')}</span><strong>{scoreText(entry.score)}</strong>
                </div>
              ))}
              <button onClick={openScores}>TÜM SKORLAR <ArrowUpRight size={17} /></button>
            </div>
          </div>
        </div>
      )}

      {screen === 'garage' && (
        <div className="panel-overlay garage-overlay">
          <div className="panel-inner garage-inner">
            <div className="panel-header">
              <div><span className="section-kicker"><span /> 15 MODEL / 5 EGZOZ</span><h2>GARAJ<span className="accent-dot">.</span></h2></div>
              <div className="panel-header-right"><div className="panel-wallet"><Coins size={20} /><strong>{creditText(profile.credits)}</strong><span>KREDİ</span></div><button className="close-button" onClick={closeOverlay} aria-label="Garajı kapat"><X size={25} /></button></div>
            </div>
            <div className="garage-grid">
              <div className="garage-stage" style={{ '--preview-color': focusedCar.color } as CSSProperties}>
                <div className="stage-circle" />
                <div className="stage-grid" />
                <div className="stage-car" key={focusedCar.id}><CarPreview car={focusedCar} /></div>
                <span className="stage-index">{focusedCar.number}<small>/ 15</small></span>
                <span className="stage-foot">{focusedCar.brand} &nbsp; / &nbsp; {focusedCar.category}</span>
              </div>
              <div className="garage-details">
                <div className="garage-current-heading">
                  <span>{focusedCar.brand} / {focusedCar.category}</span>
                  <h3>{focusedCar.name}</h3>
                  <p>{focusedCar.description}</p>
                  <div className="real-price"><span>PİYASA FİYATI</span><strong>{focusedCar.realPrice}</strong></div>
                </div>
                <div className="specs">
                  <div className="spec-row"><span>KONTROL</span><div className="spec-track"><i style={{ width: `${Math.min(100, controlValue / 1.9 * 100)}%`, background: focusedCar.color }} /></div><b>{controlValue.toFixed(2)}</b></div>
                  <div className="spec-row"><span>MAKS HIZ</span><div className="spec-track"><i style={{ width: `${Math.min(100, speedValue / 440 * 100)}%`, background: focusedCar.color }} /></div><b>{speedValue}</b></div>
                  <div className="spec-row"><span>FREN</span><div className="spec-track"><i style={{ width: `${Math.min(100, brakeValue / 200 * 100)}%`, background: focusedCar.color }} /></div><b>{brakeValue}</b></div>
                </div>
                <div className="mods-section">
                  <div className="mods-heading"><Wrench size={15} /> GELİŞTİRMELER</div>
                  {MOD_META.map(({ stat, label, hint, icon }) => {
                    const level = focusedMods[stat];
                    const maxed = level >= MAX_MOD_LEVEL;
                    const cost = upgradeCost(focusedCar, level);
                    const afford = profile.credits >= cost;
                    return (
                      <div className="mod-row" key={stat}>
                        <span className="mod-icon">{icon}</span>
                        <div className="mod-info">
                          <span>{label}</span>
                          <div className="mod-pips" title={hint}>
                            {Array.from({ length: MAX_MOD_LEVEL }, (_, i) => (
                              <i key={i} className={i < level ? 'mod-pip-on' : ''} style={i < level ? { background: focusedCar.color } : undefined} />
                            ))}
                          </div>
                        </div>
                        <button
                          className="mod-buy"
                          disabled={maxed || !afford || !focusedOwned}
                          onClick={() => buyUpgrade(focusedCar, stat)}
                          title={!focusedOwned ? 'Önce aracı satın al' : hint}
                        >
                          {maxed ? 'MAX' : `+${creditText(cost)}`}
                        </button>
                      </div>
                    );
                  })}
                </div>
                <div className="mods-section">
                  <div className="mods-heading">EGZOZ SESİ</div>
                  <div className="exhaust-grid">
                    {EXHAUSTS.map((item) => {
                      const owned = focusedMods.ownedExhaust.includes(item.id);
                      const active = focusedMods.exhaust === item.id;
                      const cost = exhaustPrice(focusedCar, item);
                      return (
                        <button
                          key={item.id}
                          className={`exhaust-chip ${active ? 'exhaust-active' : ''}`}
                          style={{ '--ex-color': item.color } as CSSProperties}
                          disabled={!focusedOwned && item.id !== 0}
                          onClick={() => buyExhaust(focusedCar, item.id)}
                          title={item.description}
                        >
                          <i />
                          <strong>{item.name}</strong>
                          <span>{owned || item.id === 0 ? (active ? 'TAKILI' : 'AÇIK') : `${creditText(cost)}`}</span>
                        </button>
                      );
                    })}
                  </div>
                  <div className="kit-row">
                    <button className={`kit-chip ${focusedMods.spoiler ? 'kit-on' : ''}`} disabled={!focusedOwned || focusedMods.spoiler > 0} onClick={() => buyKit(focusedCar, 'spoiler')}>
                      SPOILER {focusedMods.spoiler ? 'TAKILI' : creditText(spoilerCost(focusedCar))}
                    </button>
                    <button className={`kit-chip ${focusedMods.neon ? 'kit-on' : ''}`} disabled={!focusedOwned || focusedMods.neon > 0} onClick={() => buyKit(focusedCar, 'neon')}>
                      <Sparkles size={14} /> NEON {focusedMods.neon ? 'TAKILI' : creditText(neonCost(focusedCar))}
                    </button>
                  </div>
                </div>
                <div className="garage-list-label">ARAÇLAR <span>15 MODEL</span></div>
                <div className="car-list">
                  {CARS.map((car) => {
                    const owned = profile.unlocked.includes(car.id);
                    return (
                      <button className={`car-row ${focusedCar.id === car.id ? 'car-row-active' : ''}`} onClick={() => { setGarageFocusId(car.id); playSound('click'); }} key={car.id}>
                        <span className="car-row-number">{car.number}</span><span className="car-row-color" style={{ background: car.color }} />
                        <span className="car-row-name">{car.brand} {car.name}</span>
                        <span className="car-row-status">{owned ? (profile.selected === car.id ? 'SEÇİLİ' : 'AÇIK') : `${creditText(car.price)} KREDİ`}</span>
                        {owned ? <Check size={17} /> : <LockKeyhole size={15} />}
                      </button>
                    );
                  })}
                </div>
                <div className="garage-purchase">
                  {focusedOwned ? (
                    <button className="primary-button" onClick={() => startRun(focusedCar.id)}><span>BU ARAÇLA OYNA</span><span className="button-icon"><ArrowUpRight size={24} /></span></button>
                  ) : (
                    <button className="primary-button" disabled={!canAfford} onClick={() => buyCar(focusedCar)}><span>{canAfford ? `SATIN AL  /  ${creditText(focusedCar.price)}` : `${creditText(focusedCar.price - profile.credits)} KREDİ EKSİK`}</span><span className="button-icon">{canAfford ? <ArrowUpRight size={24} /> : <LockKeyhole size={20} />}</span></button>
                  )}
                  <span>{focusedOwned ? 'Seç ve hemen yola çık.' : 'Yakın geçiş yaparak kredi kazan.'}</span>
                </div>
              </div>
            </div>
          </div>
        </div>
      )}

      {screen === 'scores' && (
        <div className="panel-overlay scores-overlay">
          <div className="panel-inner scores-inner">
            <div className="panel-header">
              <div><span className="section-kicker"><span /> YEREL REKORLAR</span><h2>SKORLAR<span className="accent-dot">.</span></h2></div>
              <button className="close-button" onClick={closeOverlay} aria-label="Skorları kapat"><X size={25} /></button>
            </div>
            <p className="scores-description">En iyi beş sürüşün bu cihazda saklanır. Zirveye bir iz bırak.</p>
            {scores.length > 0 ? (
              <div className="scores-table-wrap">
                <table className="scores-table">
                  <thead><tr><th>SIRA</th><th>PUAN</th><th>ARAÇ</th><th>GEÇİŞ</th><th>TARİH</th></tr></thead>
                  <tbody>{scores.map((entry, index) => {
                    const car = CARS.find((item) => item.id === entry.carId);
                    const map = MAPS.find((item) => item.id === entry.mapId);
                    return (
                      <tr className={entry.id === lastScoreId ? 'latest-score' : ''} key={entry.id}>
                        <td>{String(index + 1).padStart(2, '0')}</td>
                        <td>{scoreText(entry.score)}</td>
                        <td><span className="score-car">{car ? `${car.brand} ${car.name}` : '—'}</span>{map && <small className="score-sub">{map.name}</small>}</td>
                        <td>{String(entry.passes).padStart(2, '0')}</td>
                        <td>{entry.date}</td>
                      </tr>
                    );
                  })}</tbody>
                </table>
              </div>
            ) : (
              <div className="empty-scores"><span>00</span><p>Henüz bir rekor yok.<br />İlk izi sen bırak.</p></div>
            )}
            <div className="scores-footer"><button className="primary-button" onClick={() => startRun()}><span>YOLA ÇIK</span><span className="button-icon"><ArrowUpRight size={24} /></span></button><button className="text-button" onClick={closeOverlay}><ArrowLeft size={18} /> GERİ DÖN</button></div>
          </div>
        </div>
      )}
    </div>
  );
}
