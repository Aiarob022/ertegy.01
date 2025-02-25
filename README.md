import { useState, useRef } from "react";
import { Button } from "@/components/ui/button";
import { Textarea } from "@/components/ui/textarea";
import { Card, CardContent } from "@/components/ui/card";
import { motion } from "framer-motion";
import { Stage, Layer, Image } from "react-konva";
import html2canvas from "html2canvas";
import { saveAs } from "file-saver";

export default function FairyTaleAnimator() {
  const [text, setText] = useState("");
  const [scenes, setScenes] = useState([]);
  const stageRef = useRef(null);

  const generateAnimation = async () => {
    if (!text.trim()) return;
    
    const extractedScenes = text.split(".").filter(scene => scene.trim().length > 0);
    setScenes(extractedScenes);
  };

  const downloadAnimation = async () => {
    if (stageRef.current) {
      const canvas = await html2canvas(stageRef.current.container());
      canvas.toBlob((blob) => {
        saveAs(blob, "animation.png");
      });
    }
  };

  return (
    <div className="flex flex-col items-center p-6">
      <Card className="w-full max-w-2xl p-4">
        <CardContent>
          <h2 className="text-xl font-bold mb-2">Ертегіңізді енгізіңіз</h2>
          <Textarea
            value={text}
            onChange={(e) => setText(e.target.value)}
            placeholder="Мұнда ертегіңізді жазыңыз..."
            className="mb-4"
          />
          <Button onClick={generateAnimation} className="mr-2">Анимация жасау</Button>
          <Button onClick={downloadAnimation} className="bg-green-500">Мультфильмді жүктеу</Button>
          {scenes.length > 0 && (
            <Stage ref={stageRef} width={500} height={300} className="mt-4 border border-gray-300">
              <Layer>
                {scenes.map((scene, index) => (
                  <motion.div 
                    key={index} 
                    initial={{ opacity: 0, x: -50 }} 
                    animate={{ opacity: 1, x: 0 }} 
                    transition={{ duration: 0.5, delay: index * 0.5 }}
                    className="absolute"
                  >
                    <Image
                      x={50 * index}
                      y={100}
                      width={100}
                      height={100}
                      src="/character.png" // Тестілік кейіпкер суреті
                    />
                  </motion.div>
                ))}
              </Layer>
            </Stage>
          )}
        </CardContent>
      </Card>
    </div>
  );
}
