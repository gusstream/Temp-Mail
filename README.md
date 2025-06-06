import React, { useEffect, useState } from "react";
import { Card, CardContent } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { formatDistanceToNow } from "date-fns";

const generateRandomEmail = (domain = "tempmail.com") => {
  const chars = "abcdefghijklmnopqrstuvwxyz1234567890";
  let email = "";
  for (let i = 0; i < 10; i++) {
    email += chars.charAt(Math.floor(Math.random() * chars.length));
  }
  return `${email}@${domain}`;
};

export default function TempMailPage() {
  const [email, setEmail] = useState("");
  const [createdAt, setCreatedAt] = useState(null);
  const [expired, setExpired] = useState(false);
  const [isPremium, setIsPremium] = useState(false);
  const [customDomain, setCustomDomain] = useState("");

  useEffect(() => {
    const domain = isPremium && customDomain ? customDomain : "tempmail.com";
    const newEmail = generateRandomEmail(domain);
    setEmail(newEmail);
    const now = new Date();
    setCreatedAt(now);

    if (!isPremium) {
      const timer = setTimeout(() => {
        setExpired(true);
      }, 2 * 60 * 60 * 1000); // 2 horas en milisegundos
      return () => clearTimeout(timer);
    }
  }, [isPremium, customDomain]);

  const handleBecomePremium = () => {
    // Aquí puedes integrar Stripe real
    setIsPremium(true);
  };

  return (
    <div className="min-h-screen flex items-center justify-center bg-gray-100 p-4">
      <Card className="w-full max-w-md shadow-xl p-4">
        <CardContent className="flex flex-col space-y-4">
          <h1 className="text-2xl font-bold text-center">Temp-Mail</h1>

          {!isPremium && (
            <Button onClick={handleBecomePremium} className="bg-yellow-500 hover:bg-yellow-600">
              Obtener Premium (Correo Permanente + Dominio Propio)
            </Button>
          )}

          {isPremium && (
            <div className="flex flex-col space-y-2">
              <label className="text-sm font-medium">Dominio personalizado:</label>
              <Input
                value={customDomain}
                onChange={(e) => setCustomDomain(e.target.value)}
                placeholder="midominio.com"
              />
              <Button onClick={() => setEmail(generateRandomEmail(customDomain))}>Generar Correo</Button>
            </div>
          )}

          {expired && !isPremium ? (
            <div className="text-center text-red-600 font-semibold">
              Este correo ha expirado.
            </div>
          ) : (
            <>
              <div className="flex items-center space-x-2">
                <Input value={email} readOnly className="flex-1" />
                <Button onClick={() => navigator.clipboard.writeText(email)}>Copiar</Button>
              </div>
              {!isPremium && (
                <div className="text-sm text-gray-500 text-center">
                  Expira en {formatDistanceToNow(new Date(createdAt.getTime() + 2 * 60 * 60 * 1000), { addSuffix: true })}
                </div>
              )}
              {isPremium && (
                <div className="text-sm text-green-600 text-center">
                  Este correo es permanente (Usuario Premium)
                </div>
              )}
            </>
          )}
        </CardContent>
      </Card>
    </div>
  );
}
