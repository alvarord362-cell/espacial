# espacial
juego
export default function SpaceGame() {
  return (
    <div className="min-h-screen bg-black text-white flex items-center justify-center p-10">
      <div className="max-w-5xl w-full rounded-3xl overflow-hidden shadow-2xl border border-cyan-500 bg-gradient-to-b from-slate-950 to-black">
        <div className="p-8 text-center">
          <h1 className="text-6xl font-black text-cyan-400 mb-4 tracking-widest animate-pulse">
            SPACE SURVIVOR 8D
          </h1>
          <p className="text-xl text-slate-300 mb-8">
            Esquiva meteoritos, gana puntos y compra escudos para sobrevivir.
          </p>
        </div>

        <GameCanvas />
      </div>
    </div>
  );
}

import { useEffect, useRef, useState } from "react";
import { Shield, Star, Rocket } from "lucide-react";

function GameCanvas() {
  const canvasRef = useRef(null);
  const [score, setScore] = useState(0);
  const [bestScore, setBestScore] = useState(0);
  const [coins, setCoins] = useState(0);
  const [shield, setShield] = useState(false);
  const [started, setStarted] = useState(false);

  useEffect(() => {
    const canvas = canvasRef.current;
    const ctx = canvas.getContext("2d");

    canvas.width = window.innerWidth * 0.9;
    canvas.height = window.innerHeight * 0.7;

    const ship = {
      x: canvas.width / 2,
      y: canvas.height - 120,
      size: 50,
      speed: 10,
    };

    let meteors = [];
    let animationFrame;

    const keys = {};

    const createMeteor = () => {
      meteors.push({
        x: Math.random() * canvas.width,
        y: -50,
        size: Math.random() * 40 + 30,
        speed: Math.random() * 5 + 4,
      });
    };

    const drawBackground = () => {
      const gradient = ctx.createLinearGradient(0, 0, 0, canvas.height);
      gradient.addColorStop(0, "#020617");
      gradient.addColorStop(1, "#000000");

      ctx.fillStyle = gradient;
      ctx.fillRect(0, 0, canvas.width, canvas.height);

      for (let i = 0; i < 120; i++) {
        ctx.fillStyle = "white";
        ctx.globalAlpha = Math.random();
        ctx.fillRect(
          Math.random() * canvas.width,
          Math.random() * canvas.height,
          2,
          2
        );
      }

      ctx.globalAlpha = 1;
    };

    const drawShip = () => {
      ctx.save();
      ctx.translate(ship.x, ship.y);

      ctx.shadowBlur = 25;
      ctx.shadowColor = "cyan";

      ctx.fillStyle = "#22d3ee";
      ctx.beginPath();
      ctx.moveTo(0, -ship.size);
      ctx.lineTo(ship.size / 2, ship.size);
      ctx.lineTo(0, ship.size / 2);
      ctx.lineTo(-ship.size / 2, ship.size);
      ctx.closePath();
      ctx.fill();

      if (shield) {
        ctx.strokeStyle = "#60a5fa";
        ctx.lineWidth = 6;
        ctx.beginPath();
        ctx.arc(0, 0, ship.size + 20, 0, Math.PI * 2);
        ctx.stroke();
      }

      ctx.restore();
    };

    const drawMeteor = (meteor) => {
      ctx.save();
      ctx.translate(meteor.x, meteor.y);

      ctx.fillStyle = "#f97316";
      ctx.shadowBlur = 20;
      ctx.shadowColor = "orange";

      ctx.beginPath();
      ctx.arc(0, 0, meteor.size, 0, Math.PI * 2);
      ctx.fill();

      ctx.restore();
    };

    const update = () => {
      drawBackground();

      if (keys["ArrowLeft"] && ship.x > 50) ship.x -= ship.speed;
      if (keys["ArrowRight"] && ship.x < canvas.width - 50)
        ship.x += ship.speed;

      if (Math.random() < 0.03) createMeteor();

      meteors.forEach((meteor, index) => {
        meteor.y += meteor.speed;

        drawMeteor(meteor);

        const dx = meteor.x - ship.x;
        const dy = meteor.y - ship.y;
        const distance = Math.sqrt(dx * dx + dy * dy);

        if (distance < meteor.size + ship.size / 2) {
          if (shield) {
            setShield(false);
            meteors.splice(index, 1);
          } else {
            setScore(0);
            setCoins(0);
          }
        }

        if (meteor.y > canvas.height + 100) {
          meteors.splice(index, 1);
          setScore((prev) => {
            const newScore = prev + 1;
            setBestScore((best) => Math.max(best, newScore));
            return newScore;
          });
          setCoins((prev) => prev + 5);
        }
      });

      drawShip();

      ctx.fillStyle = "white";
      ctx.font = "bold 28px Arial";
      ctx.fillText(`Puntos: ${score}`, 30, 50);
      ctx.fillText(`Monedas: ${coins}`, 30, 90);
      ctx.fillText(`Récord: ${bestScore}`, 30, 130);

      animationFrame = requestAnimationFrame(update);
    };

    if (started) update();

    const keyDown = (e) => {
      keys[e.key] = true;
    };

    const keyUp = (e) => {
      keys[e.key] = false;
    };

    window.addEventListener("keydown", keyDown);
    window.addEventListener("keyup", keyUp);

    return () => {
      cancelAnimationFrame(animationFrame);
      window.removeEventListener("keydown", keyDown);
      window.removeEventListener("keyup", keyUp);
    };
  }, [started, shield]);

  return (
    <div className="p-6">
      {!started && (
        <div className="text-center mb-6">
          <button
            onClick={() => setStarted(true)}
            className="bg-cyan-500 hover:bg-cyan-400 transition-all px-10 py-4 rounded-2xl text-2xl font-bold shadow-2xl"
          >
            INICIAR JUEGO
          </button>
        </div>
      )}

      <div className="flex gap-4 justify-center mb-6 flex-wrap">
        <button
          onClick={() => {
            if (coins >= 50 && !shield) {
              setCoins(coins - 50);
              setShield(true);
            }
          }}
          className="bg-blue-600 hover:bg-blue-500 px-6 py-3 rounded-2xl flex items-center gap-2 font-bold"
        >
          <Shield /> Comprar Escudo (50)
        </button>

        <div className="bg-slate-800 px-6 py-3 rounded-2xl flex items-center gap-2">
          <Rocket /> Movimiento: Flechas ← →
        </div>

        <div className="bg-yellow-500 text-black px-6 py-3 rounded-2xl flex items-center gap-2 font-bold">
          <Star /> Calidad 4K + Efectos 8D
        </div>
      </div>

      <canvas
        ref={canvasRef}
        className="rounded-3xl border-4 border-cyan-500 shadow-[0_0_60px_rgba(34,211,238,0.8)] w-full"
      />
    </div>
  );
}

