import React, { useState } from "react";
import { Button } from "@/components/ui/button";
import { Card, CardContent } from "@/components/ui/card";
import { motion } from "framer-motion";

const sampleProducts = {
  "modern_wood_neutral": [
    {
      name: "Minimal Wood Armchair",
      price: "$560",
      image: "https://images.unsplash.com/photo-1600585154340-be6161a56a0c",
      reason: "자연스러운 우드톤과 깔끔한 라인이 공간의 모던함을 살려줍니다."
    },
    {
      name: "Rattan Floor Lamp",
      price: "$220",
      image: "https://images.unsplash.com/photo-1601933470928-c7d6b2ebfc94",
      reason: "따뜻한 조명을 더해 공간에 포근한 무드를 연출할 수 있습니다."
    },
    {
      name: "Abstract Canvas Art",
      price: "$150",
      image: "https://images.unsplash.com/photo-1616627984703-d3b1c72f940d",
      reason: "모던한 분위기에 포인트를 줄 수 있는 아트 오브제입니다."
    }
  ]
};

async function analyzeImage(base64: string): Promise<string[]> {
  const response = await fetch("/api/analyzeImage", {
    method: "POST",
    headers: {
      "Content-Type": "application/json"
    },
    body: JSON.stringify({ image: base64 })
  });
  const data = await response.json();
  return data.tags;
}

export default function Home() {
  const [styleTags, setStyleTags] = useState<string[] | null>(null);
  const [uploadedImage, setUploadedImage] = useState<string | null>(null);
  const [loading, setLoading] = useState(false);

  const handleImageUpload = async (e: React.ChangeEvent<HTMLInputElement>) => {
    const file = e.target.files?.[0];
    if (!file) return;
    const reader = new FileReader();
    reader.onloadend = async () => {
      const base64 = reader.result as string;
      setUploadedImage(base64);
      setLoading(true);
      try {
        const tags = await analyzeImage(base64);
        setStyleTags(tags);
      } catch (err) {
        console.error("분석 실패:", err);
      }
      setLoading(false);
    };
    reader.readAsDataURL(file);
  };

  const handleStyleSelect = (tags: string[]) => {
    setStyleTags(tags);
  };

  const renderProductCards = () => {
    if (!styleTags) return null;
    const key = styleTags.join("_");
    const products = sampleProducts[key] || [];

    return (
      <section className="grid grid-cols-1 md:grid-cols-3 gap-8 px-8 pb-12">
        {products.map((product, index) => (
          <Card key={index} className="overflow-hidden">
            <img src={product.image} alt={product.name} className="w-full h-64 object-cover" />
            <CardContent className="p-4">
              <h3 className="text-lg font-light mb-1">{product.name}</h3>
              <p className="text-sm text-neutral-500 mb-1">{product.price}</p>
              <p className="text-xs text-neutral-400 mb-2 italic">{product.reason}</p>
              <Button variant="outline">View Details</Button>
            </CardContent>
          </Card>
        ))}
      </section>
    );
  };

  return (
    <div className="min-h-screen bg-white text-neutral-800">
      <header className="flex justify-between items-center px-6 py-4 border-b">
        <h1 className="text-2xl font-light">StyleFit</h1>
        <nav className="space-x-6 text-sm">
          <a href="#" className="hover:underline">Home</a>
          <a href="#" className="hover:underline">Products</a>
          <a href="#" className="hover:underline">About</a>
          <a href="#" className="hover:underline">Contact</a>
        </nav>
      </header>

      <section className="p-8 text-center">
        <motion.h2
          initial={{ opacity: 0, y: 20 }}
          animate={{ opacity: 1, y: 0 }}
          transition={{ duration: 0.6 }}
          className="text-3xl font-light mb-4"
        >
          당신의 공간에 딱 맞는 인테리어를 추천해드립니다.
        </motion.h2>

        {!styleTags ? (
          <div className="mt-6">
            <input type="file" accept="image/*" onChange={handleImageUpload} className="mb-4" />
            {loading && <p className="text-sm text-neutral-500 mb-2">이미지 분석 중입니다...</p>}
            <div className="text-sm text-neutral-500 mb-2">또는 스타일을 선택해주세요:</div>
            <div className="space-x-2">
              <Button onClick={() => handleStyleSelect(["modern", "wood", "neutral"])}>모던 + 우드톤</Button>
              <Button onClick={() => handleStyleSelect(["vintage", "leather"])}>빈티지 + 가죽</Button>
              <Button onClick={() => handleStyleSelect(["natural", "light"])}>내추럴 + 밝은 공간</Button>
            </div>
          </div>
        ) : (
          <motion.div
            initial={{ opacity: 0 }}
            animate={{ opacity: 1 }}
            transition={{ duration: 0.8 }}
            className="mt-6"
          >
            <p className="text-sm text-neutral-500 mb-4">분석된 스타일: {styleTags.join(", ")}</p>
            {renderProductCards()}
          </motion.div>
        )}
      </section>

      <footer className="border-t text-sm text-neutral-400 text-center py-4">
        © 2025 StyleFit. All rights reserved.
      </footer>
    </div>
  );
}
