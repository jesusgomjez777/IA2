import { useEffect, useRef, useState } from "react";
import { ElevenLabsClient } from "@elevenlabs/elevenlabs-js";

interface Props {
    texto: string;
}

export default function Asistente({
    texto
}: Props) {

    const [hablando, setHablando] = useState(false);

    const audioRef = useRef<HTMLAudioElement | null>(null);

    const asistenteRef =
        useRef<HTMLDivElement | null>(null);

    const animationRef = useRef<number | null>(null);

    const elevenlabs = new ElevenLabsClient({
        apiKey: import.meta.env.VITE_ELEVENLABS_API_KEY
    });

    useEffect(() => {
        if (!texto) return;

        reproducir();

        return () => {
            detener();
        };

    }, [texto]);

    const detener = () => {

        if (audioRef.current) {
            audioRef.current.pause();
            audioRef.current = null;
        }

        if (animationRef.current) {
            cancelAnimationFrame(animationRef.current);
        }

        setHablando(false);
    };

    const reproducir = async () => {

        try {

            detener();

            setHablando(true);

            const audioStream =
                await elevenlabs.textToSpeech.convert(
                    "hpp4J3VqNfWAUOO0d1Us",
                    {
                        text: texto,
                        modelId: "eleven_multilingual_v2",
                        outputFormat: "mp3_44100_128"
                    }
                );

            // Convertir stream a blob
            const chunks: Uint8Array[] = [];

            for await (const chunk of audioStream) {
                chunks.push(chunk);
            }

            const blob = new Blob(chunks as any, {
                type: "audio/mpeg"
            });

            const audioUrl =
                URL.createObjectURL(blob);

            const audio = new Audio(audioUrl);

            audioRef.current = audio;

            // ANALIZADOR DE AUDIO
            const audioContext =
                new AudioContext();

            const analyser =
                audioContext.createAnalyser();

            analyser.fftSize = 256;

            const source =
                audioContext.createMediaElementSource(
                    audio
                );

            source.connect(analyser);

            analyser.connect(
                audioContext.destination
            );

            const dataArray =
                new Uint8Array(
                    analyser.frequencyBinCount
                );

            const animar = () => {

                analyser.getByteFrequencyData(
                    dataArray
                );

                const promedio =
                    dataArray.reduce(
                        (a, b) => a + b,
                        0
                    ) / dataArray.length;

                const intensidad =
                    Math.min(promedio / 50, 1);

                if (asistenteRef.current) {

                    asistenteRef.current.style.transform =
                        `scale(${1 + intensidad * 0.25})`;

                    asistenteRef.current.style.boxShadow =
                        `0 0 ${
                            20 + intensidad * 40
                        }px rgba(
                            59,
                            130,
                            246,
                            ${0.4 + intensidad}
                        )`;
                }

                animationRef.current =
                    requestAnimationFrame(animar);
            };

            audio.onplay = () => {
                animar();
            };

            audio.onended = () => {

                setHablando(false);

                if (animationRef.current) {
                    cancelAnimationFrame(
                        animationRef.current
                    );
                }

                if (asistenteRef.current) {

                    asistenteRef.current.style.transform =
                        "scale(1)";

                    asistenteRef.current.style.boxShadow =
                        "none";
                }
            };

            await audio.play();

        } catch (error) {

            console.error(error);

            setHablando(false);
        }
    };

    return (
        <div
            ref={asistenteRef}
            className={`asistente ${
                hablando ? "hablando" : ""
            }`}
        >
            <div className="nucleo" />
        </div>
    );
}